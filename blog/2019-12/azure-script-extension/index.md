---
title: Using the Azure custom script extension for complex installations
description: A deep dive into the Azure custom script extension for Windows VMs
author: matthew.casperson@octopus.com
visibility: public
published: 2019-12-17
metaImage: azure-script-extension.png
bannerImage: azure-script-extension.png
bannerImageAlt: Azure custom script extensions
tags:
 - DevOps
 - Azure
---

![Azure custom script extensions](azure-script-extension.png)

When booting a virtual machine (VM) for the first time, it is often useful to run a custom script. This script may install additional software, configure the VM, or perform some other management task.

In Azure, the [custom script extension](https://docs.microsoft.com/en-au/azure/virtual-machines/extensions/custom-script-windows) provides this ability to run scripts. When Windows Azure VMs are combined with tools like Chocolatey, it becomes possible to initialize a new VM with almost any software you require.

However, there are some edge cases with Windows that you need to take into account, and in this blog post, we’ll dive into the details on performing complex installations via Azure custom script extensions.

## A simple example - configure an Windows Azure VM

Let’s start with a very simple example. The Terraform example script below configures a Windows VM with a custom script extension:

```
variable "resgroupname" {
  type = "string"
}

resource "azurerm_resource_group" "test" {
  name     = "${var.resgroupname}"
  location = "Australia East"
}

resource "azurerm_public_ip" "test" {
  name                    = "test-pip"
  location                = "${azurerm_resource_group.test.location}"
  resource_group_name     = "${azurerm_resource_group.test.name}"
  allocation_method       = "Dynamic"
  idle_timeout_in_minutes = 30
}

resource "azurerm_virtual_network" "test" {
  name                = "acctvn"
  address_space       = ["10.0.0.0/16"]
  location            = "${azurerm_resource_group.test.location}"
  resource_group_name = "${azurerm_resource_group.test.name}"
}

resource "azurerm_subnet" "test" {
  name                 = "acctsub"
  resource_group_name  = "${azurerm_resource_group.test.name}"
  virtual_network_name = "${azurerm_virtual_network.test.name}"
  address_prefix       = "10.0.2.0/24"
}

resource "azurerm_network_interface" "test" {
  name                = "acctni"
  location            = "${azurerm_resource_group.test.location}"
  resource_group_name = "${azurerm_resource_group.test.name}"

  ip_configuration {
    name                          = "testconfiguration1"
    subnet_id                     = "${azurerm_subnet.test.id}"
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = "${azurerm_public_ip.test.id}"
  }
}

resource "azurerm_storage_account" "test" {
  name                     = "${var.resgroupname}"
  resource_group_name      = "${azurerm_resource_group.test.name}"
  location                 = "${azurerm_resource_group.test.location}"
  account_tier             = "Standard"
  account_replication_type = "LRS"
}

resource "azurerm_storage_container" "test" {
  name                  = "vhds"
  resource_group_name   = "${azurerm_resource_group.test.name}"
  storage_account_name  = "${azurerm_storage_account.test.name}"
  container_access_type = "private"
}

resource "azurerm_virtual_machine" "test" {
  name                  = "acctvm"
  location              = "${azurerm_resource_group.test.location}"
  resource_group_name   = "${azurerm_resource_group.test.name}"
  network_interface_ids = ["${azurerm_network_interface.test.id}"]
  vm_size               = "Standard_D2_v3"

  storage_image_reference {
    publisher = "MicrosoftWindowsServer"
    offer     = "WindowsServer"
    sku       = "2019-Datacenter"
    version   = "latest"
  }

  storage_os_disk {
    name          = "osdisk"
    vhd_uri       = "${azurerm_storage_account.test.primary_blob_endpoint}${azurerm_storage_container.test.name}/osdisk.vhd"
    caching       = "ReadWrite"
    create_option = "FromImage"
  }

  os_profile {
    computer_name  = "hostname"
    admin_username = "testadmin"
    admin_password = "passwordoeshere"
  }

  os_profile_windows_config {
    enable_automatic_upgrades = false
    provision_vm_agent = true
  }
}

resource "azurerm_virtual_machine_extension" "test" {
  name                 = "hostname"
  location             = "${azurerm_resource_group.test.location}"
  resource_group_name  = "${azurerm_resource_group.test.name}"
  virtual_machine_name = "${azurerm_virtual_machine.test.name}"
  publisher            = "Microsoft.Compute"
  type                 = "CustomScriptExtension"
  type_handler_version = "1.9"

  protected_settings = <<PROTECTED_SETTINGS
    {
      "commandToExecute": "powershell.exe -Command \"./chocolatey.ps1; exit 0;\""
    }
  PROTECTED_SETTINGS

  settings = <<SETTINGS
    {
        "fileUris": [
          "https://gist.githubusercontent.com/mcasperson/c815ac880df481418ff2e199ea1d0a46/raw/5d4fc583b28ecb27807d8ba90ec5f636387b00a3/chocolatey.ps1"
        ]
    }
  SETTINGS
}
```

The important part of this script is the  `azurerm_virtual_machine_extension` resource. In the `settings` field, we have a JSON blob listing scripts to download in the `fileUris` array, and in the `protected_settings` field, we have another JSON blob with a `commandToExecute` string defining the entry point to the script we are going to run.

In this example we are downloading a PS1 PowerShell script file from a GitHub Gist that installs Chocolatey and then installs Notepad++ with the Chocolatey client:

```
Set-ExecutionPolicy Bypass -Scope Process -Force
iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
choco install notepadplusplus -y
```

The downloaded script file is then run by the string assigned to the `commandToExecute` field, with an `exit 0` to ensure that the script extension registers that our script ran successfully:

```
 "commandToExecute": "powershell.exe -Command \"./chocolatey.ps1; exit 0;\""
```

:::hint
Trying to encode a PowerShell script into the `commandToExecute` JSON string quickly becomes unmanageable. Downloading scripts using the `fileUris` is a much better solution, and the scripts can be hosted in Azure blob storage for better security if needed.
:::

This example is quite straight forward, and for simple software installations, this is all we need. Unfortunately, not all software is this easy to install.

## Automating the install of SQL Server on an Azure VM

To see where this example breaks down, we will try installing Microsoft SQL Server Express:

```
Set-ExecutionPolicy Bypass -Scope Process -Force
iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
choco install sql-server-express -y
```

SQL Server is obviously a more complex piece of software than Notepad++, and while attempting to install it, we run into some errors. Here in the Chocolatey logs, we can see that SQL Server failed to install:

```
2019-11-06 05:47:59,622 2240 [WARN ] - WARNING: May not be able to find 'C:\windows\TEMP\chocolatey\sql-server-express\2017.20190916\SQLEXPR\setup.exe'. Please use full path for executables.
2019-11-06 05:47:59,751 2240 [ERROR] - ERROR: Exception calling "Start" with "0" argument(s): "The system cannot find the file specified"
```

This error occurs because the [System account](https://www.powershellmagazine.com/2014/04/30/understanding-azure-custom-script-extension/) that runs the custom script won’t work with the SQL Server install. We need a way to run the installation as a regular administrator account.

## Running a PowerShell script under a new account

PowerShell offers a convenient solution to running code as a different user. By calling `Invoke-Command`, we can execute a script block as a user of our choosing on the local (or remote if necessary) VM.

The code below shows how we can build up a credentials object and pass it to the `Invoke-Command` command to execute the Chocolaty install as the administrator user:

:::hint
The username must be in the format `machinename\username` for this command to work correctly.
:::

```
$securePassword = ConvertTo-SecureString 'passwordgoeshere' -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential 'hostname\testadmin', $securePassword
Invoke-Command -ScriptBlock {choco install sql-server-express -y} -ComputerName hostname -Credential $credential
```

This *almost* works, but now we get this error:

```
Inner exception type: System.InvalidOperationException
       Message:
               There was an error generating the XML document.
       HResult : 0x80131509
```

Unfortunately, the default call to `Invoke-Command` doesn’t grant enough permissions for the installation to complete. We have run into the double hop issue, which results in the exception shown above.

## Supporting PowerShell double hops

Executing code on a remote machine, or executing “remote” code on the same machine, is considered to be the first hop. Having that code go on to access another machine is considered a [PowerShell double hop](https://docs.microsoft.com/en-us/powershell/scripting/learn/remoting/ps-remoting-second-hop?view=powershell-6), and this second hop is prevented by default when using `Invoke-Command`. There are other calls that are also considered to be a second hop, which our SQL Server installation ran into.

To allow the SQL Server installation to complete, we need to allow this double hop to take place. We do this by taking advantage of CredSSP authentication.

The first step is to enable our machine to take on the `Server` role, which means it can accept credentials from a client:

```
Enable-WSManCredSSP -Role Server -Force
```

We then need to enable our machine to take on the `Client` role, meaning it can send credentials to the server. We enable both the client and server roles because we are using `Invoke-Command` to run the command on the same machine:

```
Enable-WSManCredSSP -Role Client -DelegateComputer * -Force
```

Because the account we are running the code as is a local account (as opposed to a domain account), we need to allow the use of NTLM accounts. This is done by setting a registry key:

```
New-Item -Path HKLM:\SOFTWARE\Policies\Microsoft\Windows\CredentialsDelegation -Name AllowFreshCredentialsWhenNTLMOnly -Force
New-ItemProperty -Path HKLM:\SOFTWARE\Policies\Microsoft\Windows\CredentialsDelegation\AllowFreshCredentialsWhenNTLMOnly -Name 1 -Value * -PropertyType String
```

With these changes in place we can run the Chocolaty install using CredSSP authentication. This enables the double hop, and our SQL Server installation will complete successfully:

```
$securePassword = ConvertTo-SecureString 'passwordgoeshere' -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential 'hostname\testadmin', $securePassword
Invoke-Command -Authentication CredSSP -ScriptBlock {choco install sql-server-express -y} -ComputerName hostname -Credential $credential
```

The complete script has been saved in this [Gist](https://gist.githubusercontent.com/mcasperson/6b87b519bd0ab1d093e697b33938ed3b/raw/b635114f552ea230e08828c63274316323799386/chocolatey.ps1).

## Conclusion

While most scripts will run as expected with the Azure custom script extension, there are times when you need to run scripts as an admin user. In this post, we saw how to take advantage of CredSSP authentication to ensure that scripts run with the highest privileges required to install some complex applications like SQL Server.
