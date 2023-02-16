---
title: Deprecating Azure VM Extension for Tentacle
description: Find out why the Azure VM Extension is deprecated and the recommended way to deploy Tentacle Windows VMs in the future.
author: veo.chen@octopus.com
visibility: private
published: 2023-03-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: Deprecating Azure VM Extension for Tentacle
isFeatured: false
tags: 
  - Automation
  - Azure
---

The [Azure VM Extension for Tentacle](https://github.com/OctopusDeploy/AzureVMExtension) was deprecated back in 2021 but never removed from the marketplace. It is no longer fully compatible with newer versions of Tentacle. We'll be removing it in the near future. This post walks you through a few alternatives in a post-Azure-VM-Extension world. Give this a read if your workflow is impacted. Note: the VM Extension only applies to Windows VMs in Azure.

## The Problem

In [https://octopus.com/blog/tentacle-dotnet-version-change](https://octopus.com/blog/tentacle-dotnet-version-change) we explained that we're dropping support for older .NET versions in Tentacle. This means Windows VMs without .NET 4.8 runtime will no longer be able to run the latest version of Tentacle.

As of 2022 there were still a few thousand Windows VMs deployed by the Azure VM Extension. We deprecated the extension a couple years ago, as Azure was moving towards ARM templates and script extensions. As a result, the extension no longer works when deploying to VMs with older OS versions. We're also going to remove the extension from the marketplace in the near future as it's overstayed its deprecation period.

## How to provision Tentacle VMs going forward

We recommend using [ARM templates](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/overview) and [Powershell DSC Extensions](https://learn.microsoft.com/en-us/azure/virtual-machines/extensions/dsc-windows) to deploy Tentacles from now on.

Below we outline the basic steps to creating such a deployment.

### Tentacle Version & .NET Version

Regardless of the provisioning method, Tentacle from `6.3` onwards requires the .NET 4.8 runtime to run on Windows. We recommend OS versions that have .NET 4.8 installed by default. If not, make sure it is installed on your VM using the ARM template or the DSC config.

Alternatively, you can lock the Tentacle version to `6.2` which is compatible with most old .NET framework versions. Once the version is locked, you can use the DSC config to install a `6.2` Tentacle and ignore newer versions. An example DSC config is shown in the next section.

- See [.NET Framework & Windows OS versions](https://learn.microsoft.com/en-us/dotnet/framework/migration-guide/versions-and-dependencies#net-framework-48) for whether .NET 4.8 is or can be installed on a given OS version.
- See [Tentacle versioning and when to update](https://octopus.com/blog/tentacle-versioning) for how to lock your Tentacle version.

### Prepare the DSC Extension

Octopus Deploy offers a [DSC module](https://github.com/OctopusDeploy/OctopusDSC) that can be used to deploy Tentacles. As explained in [Installing the Tentacle via DSC in an ARM template](https://octopus.com/docs/infrastructure/deployment-targets/tentacle/windows/azure-virtual-machines/via-an-arm-template-with-dsc), the first step is to create a zip file containing the DSC source and a DSC config. The config file can be kept simple or modifed to accept parameters pertaining to your workflow. For example, here we add a `CommunicationMode` parameter to deploy Tentacles in different modes.

```powershell
configuration OctopusTentacle
{
    param ($ApiKey, $OctopusServerUrl, $Environments, $Roles, $ServerPort, $CommunicationMode)

    Import-DscResource -Module OctopusDSC

    Node "localhost"
    {
        cTentacleAgent OctopusTentacle
        {
            Ensure = "Present"
            State = "Started"

            # Tentacle instance name. Leave it as 'Tentacle' unless you have more
            # than one instance
            Name = "Tentacle"

            # Registration - all parameters required
            ApiKey = $ApiKey
            OctopusServerUrl = $OctopusServerUrl
            Environments = $Environments
            Roles = $Roles

            # How Tentacle will communicate with the server
            CommunicationMode = $CommunicationMode
            ServerPort = $ServerPort

            # Where deployed applications will be installed by Octopus
            DefaultApplicationDirectory = "C:\Applications"

            # Where Octopus should store its working files, logs, packages etc
            TentacleHomeDirectory = "C:\Octopus"
        }
    }
}
```

If you're locking Tentacle to version `6.2`, update the DSC config to only download `6.2` during installation.

```powershell
configuration OctopusTentacle
{
    param ($ApiKey, $OctopusServerUrl, $Environments, $Roles, $ServerPort, $CommunicationMode)

    Import-DscResource -Module OctopusDSC

    Node "localhost"
    {
        cTentacleAgent OctopusTentacle
        {
            Ensure = "Present"
            State = "Started"

            # Tentacle instance name. Leave it as 'Tentacle' unless you have more
            # than one instance
            Name = "Tentacle"

            # Registration - all parameters required
            ApiKey = $ApiKey
            OctopusServerUrl = $OctopusServerUrl
            Environments = $Environments
            Roles = $Roles

            # How Tentacle will communicate with the server
            CommunicationMode = $CommunicationMode
            ServerPort = $ServerPort

            # Where deployed applications will be installed by Octopus
            DefaultApplicationDirectory = "C:\Applications"

            # Where Octopus should store its working files, logs, packages etc
            TentacleHomeDirectory = "C:\Octopus"

            # Lock Tentacle to 6.2 to support older runtimes
            tentacleDownloadUrl = "https://download.octopusdeploy.com/octopus/Octopus.Tentacle.6.2.277.msi"
            tentacleDownloadUrl64 = "https://download.octopusdeploy.com/octopus/Octopus.Tentacle.6.2.277-x64.msi"
        }
    }
}
```

### Prepare the ARM template

#### `location` of the DSC Extension

The DSC Extension needs to be deployed to the same `location` as the VM in order to find it. You can use the same expression for both the VM and the extension's location such as `[resourceGroup().location]`, or use a parameter such as `[parameters('vmLocation')]`.

When defining the location as a parameter, the value needs to be the id of the region, e.g. `australiacentral` or `westus2`. One way to see the list of all regions and their ids is using the command `az account list-locations -o table`. If it's an existing VM, you can also find this by going to the VM page and looking at `JSON View`.

#### Using ARM template to provision Tentacle VM

The easiest option is to deploy Tentacle along with your VM and other related resources. Everything is defined together and deployed at the same time, and can be easily redeployed if needed.

Here's what the ARM template might look like.


```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmAdminUsername": {
      "type": "string",
      "metadata": {
        "description": "Admin username for the Virtual Machine."
      }
    },
    "vmAdminPassword": {
      "type": "securestring",
      "metadata": {
        "description": "Admin password for the Virtual Machine."
      }
    },
    "vmDnsName": {
      "type": "string",
      "metadata": {
        "description": "Unique DNS Name for the Public IP used to access the Virtual Machine."
      }
    },
    "vmSize": {
      "defaultValue": "Standard_D2_v2",
      "type": "string",
      "metadata": {
        "description": "Size of the Virtual Machine"
      }
    },
    "tentacleOctopusServerUrl": {
      "type": "string",
      "metadata": {
        "description": "The URL of the octopus server with which to register"
      }
    },
    "tentacleApiKey": {
      "type": "securestring",
      "metadata": {
        "description": "The Api Key to use to register the Tentacle with the server"
      }
    },
    "tentacleCommunicationMode": {
      "defaultValue": "Listen",
      "allowedValues": [
        "Listen",
        "Poll"
      ],
      "type": "string",
      "metadata": {
        "description": "The type of Tentacle - whether the Tentacle listens for requests from server, or actively polls the server for requests"
      }
    },
    "tentaclePort": {
      "defaultValue": 10933,
      "minValue": 0,
      "maxValue": 65535,
      "type": "int",
      "metadata": {
        "description": "The port on which the Tentacle should listen, when CommunicationMode is set to Listen, or the port on which to poll the server, when CommunicationMode is set to Poll. By default, Tentacle's listen on 10933 and poll the Octopus Server on 10943."
      }
    },
    "tentacleRoles": {
      "type": "string",
      "metadata": {
        "description": "A comma delimited list of roles to apply to the Tentacle"
      }
    },
    "tentacleEnvironments": {
      "type": "string",
      "metadata": {
        "description": "A comma delimited list of environments in which the Tentacle should be placed"
      }
    }
  },
  "variables": {
    "namespace": "octopus",
    "location": "[resourceGroup().location]",
    "tags": {
      "vendor": "Octopus Deploy",
      "description": "Example deployment of Octopus Tentacle to a Windows Server."
    },
    "diagnostics": {
      "storageAccount": {
        "name": "[concat('diagnostics', uniquestring(resourceGroup().id))]"
      }
    },
    "networkSecurityGroupName": "[concat(variables('namespace'), '-nsg')]",
    "publicIPAddressName": "[concat(variables('namespace'), '-publicip')]",
    "vnet": {
      "name": "[concat(variables('namespace'), '-vnet')]",
      "addressPrefix": "10.0.0.0/16",
      "subnet": {
        "name": "[concat(variables('namespace'), '-subnet')]",
        "addressPrefix": "10.0.0.0/24"
      }
    },
    "nic": {
      "name": "[concat(variables('namespace'), '-nic')]",
      "ipConfigName": "[concat(variables('namespace'), '-ipconfig')]"
    },
    "vmName": "[concat(variables('namespace'),'-vm')]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2021-04-01",
      "name": "[variables('diagnostics').storageAccount.name]",
      "location": "[variables('location')]",
      "tags": {
        "vendor": "[variables('tags').vendor]",
        "description": "[variables('tags').description]"
      },
      "kind": "Storage",
      "sku": {
        "name": "Standard_LRS"
      },
      "properties": {}
    },
    {
      "type": "Microsoft.Network/networkSecurityGroups",
      "apiVersion": "2021-02-01",
      "name": "[variables('networkSecurityGroupName')]",
      "location": "[variables('location')]",
      "tags": {
        "vendor": "[variables('tags').vendor]",
        "description": "[variables('tags').description]"
      },
      "properties": {
        "securityRules": [
          {
            "name": "allow_listening_tentacle",
            "properties": {
              "description": "Allow inbound Tentacle connection",
              "protocol": "Tcp",
              "sourcePortRange": "*",
              "destinationPortRange": "10933",
              "sourceAddressPrefix": "*",
              "destinationAddressPrefix": "*",
              "access": "Allow",
              "priority": 123,
              "direction": "Inbound"
            }
          }
        ]
      }
    },
    {
      "type": "Microsoft.Network/publicIPAddresses",
      "name": "[variables('publicIPAddressName')]",
      "apiVersion": "2021-02-01",
      "location": "[variables('location')]",
      "tags": {
        "vendor": "[variables('tags').vendor]",
        "description": "[variables('tags').description]"
      },
      "properties": {
        "publicIPAllocationMethod": "Dynamic",
        "dnsSettings": {
          "domainNameLabel": "[parameters('vmDnsName')]"
        }
      }
    },
    {
      "type": "Microsoft.Network/virtualNetworks",
      "name": "[variables('vnet').name]",
      "apiVersion": "2021-02-01",
      "location": "[variables('location')]",
      "tags": {
        "vendor": "[variables('tags').vendor]",
        "description": "[variables('tags').description]"
      },
      "dependsOn": [
        "[concat('Microsoft.Network/networkSecurityGroups/', variables('networkSecurityGroupName'))]"
      ],
      "properties": {
        "addressSpace": {
          "addressPrefixes": [
            "[variables('vnet').addressPrefix]"
          ]
        },
        "subnets": [
          {
            "name": "[variables('vnet').subnet.name]",
            "properties": {
              "addressPrefix": "[variables('vnet').subnet.addressPrefix]",
              "networkSecurityGroup": {
                "id": "[resourceId('Microsoft.Network/networkSecurityGroups', variables('networkSecurityGroupName'))]"
              }
            }
          }
        ]
      }
    },
    {
      "type": "Microsoft.Network/networkInterfaces",
      "name": "[variables('nic').name]",
      "apiVersion": "2021-02-01",
      "location": "[variables('location')]",
      "tags": {
        "vendor": "[variables('tags').vendor]",
        "description": "[variables('tags').description]"
      },
      "dependsOn": [
        "[concat('Microsoft.Network/virtualNetworks/', variables('vnet').name)]",
        "[concat('Microsoft.Network/publicIPAddresses/', variables('publicIPAddressName'))]",
        "[concat('Microsoft.Network/networkSecurityGroups/', variables('networkSecurityGroupName'))]"
      ],
      "properties": {
        "ipConfigurations": [
          {
            "name": "[variables('nic').ipConfigName]",
            "properties": {
              "privateIPAllocationMethod": "Dynamic",
              "publicIPAddress": {
                "id": "[resourceId('Microsoft.Network/publicIPAddresses', variables('publicIPAddressName'))]"
              },
              "subnet": {
                "id": "[concat(resourceId('Microsoft.Network/virtualNetworks', variables('vnet').name), '/subnets/', variables('vnet').subnet.name)]"
              }
            }
          }
        ]
      }
    },
    {
      "type": "Microsoft.Compute/virtualMachines",
      "name": "[variables('vmName')]",
      "apiVersion": "2021-04-01",
      "location": "[variables('location')]",
      "tags": {
        "vendor": "[variables('tags').vendor]",
        "description": "[variables('tags').description]"
      },
      "dependsOn": [
        "[concat('Microsoft.Storage/storageAccounts/', variables('diagnostics').storageAccount.name)]",
        "[concat('Microsoft.Network/networkInterfaces/', variables('nic').name)]"
      ],
      "properties": {
        "hardwareProfile": {
          "vmSize": "[parameters('vmSize')]"
        },
        "osProfile": {
          "computerName": "[variables('vmName')]",
          "adminUsername": "[parameters('vmAdminUsername')]",
          "adminPassword": "[parameters('vmAdminPassword')]"
        },
        "storageProfile": {
          "imageReference": {
            "publisher": "MicrosoftWindowsServer",
            "offer": "WindowsServer",
            "sku": "2016-Datacenter",
            "version": "latest"
          },
          "osDisk": {
            "createOption": "FromImage",
            "caching": "ReadWrite",
            "managedDisk": {
              "storageAccountType": "Standard_LRS"
            }
          }
        },
        "networkProfile": {
          "networkInterfaces": [
            {
              "id": "[resourceId('Microsoft.Network/networkInterfaces', variables('nic').name)]"
            }
          ]
        },
        "diagnosticsProfile": {
          "bootDiagnostics": {
            "enabled": true,
            "storageUri": "[concat('http://', variables('diagnostics').storageAccount.name, '.blob.core.windows.net')]"
          }
        }
      }
    },
    {
      "type": "Microsoft.Compute/virtualMachines/extensions",
      "name": "[concat(variables('vmName'),'/dscExtension')]",
      "apiVersion": "2021-04-01",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
      ],
      "properties": {
        "publisher": "Microsoft.Powershell",
        "type": "DSC",
        "typeHandlerVersion": "2.77",
        "autoUpgradeMinorVersion": true,
        "forceUpdateTag": "2",
        "settings": {
          "configuration": {
              "url": "https://url-to-storage/OctopusTentacle.zip",
              "script": "OctopusTentacle.ps1",
              "function": "OctopusTentacle"
          },
          "configurationArguments": {
              "ApiKey": "[parameters('tentacleApiKey')]",
              "OctopusServerUrl": "[parameters('tentacleOctopusServerUrl')]",
              "Environments": "[parameters('tentacleEnvironments')]",
              "Roles": "[parameters('tentacleRoles')]",
              "ServerPort": "[parameters('tentaclePort')]",
              "CommunicationMode":"[parameters('tentacleCommunicationMode')]"
          }
        },
        "protectedSettings": null
      }
    }
  ]
}
```

The network group security rule in this example allows for Listening Tentacles to be talked to via the default port 10933. Polling Tentacles don't require an open port and can be deployed without one.

For more examples, see [Installing the Tentacle via DSC in an ARM template](https://octopus.com/docs/infrastructure/deployment-targets/tentacle/windows/azure-virtual-machines/via-an-arm-template-with-dsc).

#### Use ARM template to install Tentacle onto existing VM

It's also possible to use an ARM template to deploy Tentacle onto an existing VM. The extension needs to be deployed to the same region as the VM in order to find it. Therefore we need to provide both the name and location of the VM.

Here's what the ARM template might look like.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "type": "string",
      "metadata": {
        "description": "Name of the Virtual Machine to run the extension on"
      }
    },
    "vmLocation": {
        "type": "string",
      "metadata": {
        "description": "Region id of the Virtual Machine, e.g. westus2"
      }
    },
    "tentacleOctopusServerUrl": {
      "type": "string",
      "metadata": {
        "description": "The URL of the octopus server with which to register"
      }
    },
    "tentacleApiKey": {
      "type": "securestring",
      "metadata": {
        "description": "The Api Key to use to register the Tentacle with the server"
      }
    },
    "tentacleCommunicationMode": {
      "defaultValue": "Listen",
      "allowedValues": ["Listen", "Poll"],
      "type": "string",
      "metadata": {
        "description": "The type of Tentacle - whether the Tentacle listens for requests from server, or actively polls the server for requests"
      }
    },
    "tentaclePort": {
      "defaultValue": 10933,
      "minValue": 0,
      "maxValue": 65535,
      "type": "int",
      "metadata": {
        "description": "The port on which the Tentacle should listen, when CommunicationMode is set to Listen, or the port on which to poll the server, when CommunicationMode is set to Poll. By default, Tentacle's listen on 10933 and poll the Octopus Server on 10943."
      }
    },
    "tentacleRoles": {
      "type": "string",
      "metadata": {
        "description": "A comma delimited list of roles to apply to the Tentacle"
      }
    },
    "tentacleEnvironments": {
      "type": "string",
      "metadata": {
        "description": "A comma delimited list of environments in which the Tentacle should be placed"
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.Compute/virtualMachines/extensions",
      "name": "[concat(parameters('vmName'),'/dscTentacleExtension')]",
      "apiVersion": "2021-04-01",
      "location": "[parameters('vmLocation')]",
      "properties": {
        "publisher": "Microsoft.Powershell",
        "type": "DSC",
        "typeHandlerVersion": "2.77",
        "autoUpgradeMinorVersion": true,
        "forceUpdateTag": "2",
        "settings": {
          "configuration": {
            "url": "https://url-to-storage/OctopusTentacle.zip",
            "script": "OctopusTentacle.ps1",
            "function": "OctopusTentacle"
          },
          "configurationArguments": {
            "ApiKey": "[parameters('tentacleApiKey')]",
            "OctopusServerUrl": "[parameters('tentacleOctopusServerUrl')]",
            "Environments": "[parameters('tentacleEnvironments')]",
            "Roles": "[parameters('tentacleRoles')]",
            "ServerPort": "[parameters('tentaclePort')]",
            "CommunicationMode":"[parameters('tentacleCommunicationMode')]"
          }
        },
        "protectedSettings": null
      }
    }
  ]
}
```

### Deploy the ARM template

To deploy the ARM template you can do it

- [from Azure Portal](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/quickstart-create-templates-use-the-portal)
- [using Azure CLI](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/deploy-cli)
- or [using Octopus](https://octopus.com/docs/runbooks/runbook-examples/azure/resource-groups)

## Next Steps

1. Shortly after this post, we're going to re-publish Tentacle `6.3` to our downloads page and `Chocolatey`. We've had to pull the latest Tentacle from these sources because of the Azure VM Extension issues described above. If you start having issues with the extension, this is likely why.
1. We're going to remove the Azure VM Extension from the marketplace at the end of March 2023, thus completing the deprecation process.

## Conclusion

With the Azure VM Extension going away, it is now recommended to use ARM templates and DSC extensions to deploy your Windows Tentacle VMs in Azure. This should give you more control over how you deploy your VMs and Tentacles. It will also allow us to update Tentacle with more confidence moving forwards so we can bring you a more robust and smooth deployment experience.

Happy deployments!
