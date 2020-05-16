---
title: Linux worker for PowerShell templates
description: Using a Linux worker to run Step Templates written in PowerShell
author: shawn.sesna@octopus.com
visibility: private
published: 2021-05-20
metaImage: 
bannerImage: 
tags:
 - 
 - 
---

Octopus Deploy started as a tool for deploying .NET applications.  As such, the vast majority of our step templates are written in PowerShell.  With the introduction of the [Linux Tentacle](https://octopus.com/docs/infrastructure/deployment-targets/linux/tentacle) there have been a handful of step templates that have been converted to Bash versions.  With PowerShell Core, converting existing templates to Bash isn't necessary*!  In this post, I demonstrate how you can use existing Octopus Deploy step templates on a Linux worker with PowerShell Core.

*There are some exceptions to this such as Windows-specific templates (Windows services, IIS, etc...)

## Creating the ARM template
The first thing I'll need for this is a Linux machine to work with.  For grins and giggles, I decided to use a Ubuntu Linux VM in Azure spun up from a Runbook.  

### Azure Resource Manager (ARM) Template
Creating a VM in Azure is dead easy using ARM templates.  Microsoft did a fantastic job of making the generation of an ARM template quick and easy.  Simply go through the process of defining how you want your VM configured and export the template at the end!  It's that simple!

![](azure-arm-template-export.png)

### ARM template parameters
The specific selections that you make when generating your ARM template aren't important as you can parameterize everything and control them with Octopus Deploy variables.  In the above image, items like `Virtual machine name`, `Username`, and `OS disk type` will all be parameters in our template to make it more generic.  This will make more sense when creating our Runbook.

## Octopus Deploy Runbook
[Operations Runbooks](https://octopus.com/docs/operations-runbooks) is one of Octopus Deploys more recent features.  Runbooks allow you to harness the power of Octopus Deploy to perform operational tasks such as restoring a database or recycling an IIS App Pool.  Unlike deployments, Runbooks are designed to be run in any environment at any time and aren't constrained by Lifecycles (other than they can only be executed in the Environments your application uses).  In this case, we're going to utilize Runbooks to spin up our Linux VM!

### Creating the Runbook
A Runbook exists inside your Octopus Deploy project, under Operations.

![](octopus-project-runbook.png)

After clicking on Runbooks, click on the **ADD RUNBOOK** button

![](octopus-project-add-runbook.png)

To get our Linux Worker VM spun up, our Runbook will consist of
- Creating a new Azure Resource Group
- Converting our Cloud Init Script to a base64 string (more on this later)
- Running an ARM template
- Wait for the VM to be available
- Perform a health check

This post is more focused on demonstrating Linux with PowerShell core, so we're not going to go into too much detail regarding runbooks.

#### Create a resource group
Creating a new resource group is optional and is mostly meant for tidiness and ease of deleting the resources created when executing an ARM template.  Using an existing resource group is perfectly fine.  Using a Run an Azure Script step, you can quickly create a resource group with a script

```PS
$resourceGroupName = $OctopusParameters["Azure.Worker.ResourceGroup.Name"]
$resourceGroupLocation = $OctopusParameters["Azure.Location.Abbr"]

Try {
    Get-AzureRmResourceGroup -Name $resourceGroupName
    $createResourceGroup = $false
} Catch {
    $createResourceGroup = $true
}

if ($createResourceGroup -eq $true){
    New-AzureRmResourceGroup -Name $resourceGroupName -Location $resourceGroupLocation
}
```

#### Converting the Cloud Init script to a base64 string
ARM templates have two different ways to execute intit scripts
- [Custom Script Extension](https://docs.microsoft.com/en-us/azure/virtual-machines/extensions/custom-script-windows):
Using the Custom Script Extension, you can supply a Uri to a file (such as a [Gist](https://gist.github.com/discover)) that contains the script to include.  The advantage of this method is the ability to source control the script file.
- CustomData parameter (this post uses this method):
The CustomData Parameter allows us to include the script as a string and pass it in as a parameter.  When provisioning Windows VMs, this value can be a string.  For Linux, this value must first be Base64 encoded, which can easily be done within a Run a script task

For this example I want my new VM to do the following
- Install Linux Tentacle
- Install PowerShell Core
- Configure Tentacle and register itself of to my cloud instance

##### Install Linux Tentacle
This portion of the script will cover the installation of the tentacle.  If you've read any of our other blog posts about Linux Tentacle, it should look familiar :)

```bash
serverUrl="#{Global.Base.Url}"   # The url of your Octous server
thumbprint="#{Global.Server.Thumbprint}"       # The thumbprint of your Octopus Server
apiKey="#{Global.Api.Key}"           # An Octopus Server api key with permission to add machines
name="#{Octopus.Space.Name}-#{Octopus.Environment.Name}"      # The name of the Tentacle at is will appear in the Octopus portal
publicHostName="#{Azure.Worker.DNS.Prefix}.centralus.cloudapp.azure.com"      # The url to the tentacle
workerPoolName="#{Azure.Worker.Pool.Name}"
configFilePath="/etc/octopus/default/tentacle-default.config"
applicationPath="/home/Octopus/Applications/"
spaceName="#{Octopus.Space.Name}"

sudo apt install --no-install-recommends gnupg curl ca-certificates apt-transport-https && \
curl -sSfL https://apt.octopus.com/public.key | sudo apt-key add - && \
sudo sh -c "echo deb https://apt.octopus.com/ stable main > /etc/apt/sources.list.d/octopus.com.list" && \
sudo apt update && sudo apt install tentacle -y
```

##### Install PowerShell Core
This bit will install PowerShell Core onto Ubuntu (see [Microsofts documentation](https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell-core-on-linux?view=powershell-7) for more information)
```bash
# Download the Microsoft repository GPG keys
wget -q https://packages.microsoft.com/config/ubuntu/18.04/packages-microsoft-prod.deb

# Register the Microsoft repository GPG keys
sudo dpkg -i packages-microsoft-prod.deb

# Update the list of products
sudo apt-get update

# Enable the "universe" repositories
sudo add-apt-repository universe

# Install PowerShell
sudo apt-get install -y powershell
```

##### Configure Tentacle and register itself of to my cloud instance
Lastly we need the script to configure the Tentacle and then register itself to our instance.  This script uses variables that were defined in our Installation section above.
```bash
sudo /opt/octopus/tentacle/Tentacle create-instance --config "$configFilePath"
sudo /opt/octopus/tentacle/Tentacle new-certificate --if-blank
sudo /opt/octopus/tentacle/Tentacle configure --port 10933 --noListen False --reset-trust --app "$applicationPath"
sudo /opt/octopus/tentacle/Tentacle configure --trust $thumbprint
echo "Registering the Tentacle $name as a worker with server $serverUrl in $workerPoolName"
sudo /opt/octopus/tentacle/Tentacle register-worker --server "$serverUrl" --apiKey "$apiKey" --name "$name" --space "$spaceName" --publicHostName "$publicHostName" --workerpool "$workerPoolName"
# Install and start the service
sudo /opt/octopus/tentacle/Tentacle service --install --start
```

With our script set up, we need to convert it to a Base64 string, then set it to an ouput variable so it can be used in the ARM template.  Here's the whole thing put together in a Run a Script step named `Convert cloud init script`

```PS
# Define cloud init script
$cloudInitScript = @'
#!/bin/bash

# Install Octpous listening tentacle
serverUrl="#{Global.Base.Url}"   # The url of your Octous server
thumbprint="#{Global.Server.Thumbprint}"       # The thumbprint of your Octopus Server
apiKey="#{Global.Api.Key}"           # An Octopus Server api key with permission to add machines
name="#{Octopus.Space.Name}-#{Octopus.Environment.Name}"      # The name of the Tentacle at is will appear in the Octopus portal
publicHostName="#{Azure.Worker.DNS.Prefix}.centralus.cloudapp.azure.com"      # The url to the tentacle
workerPoolName="#{Azure.Worker.Pool.Name}"
configFilePath="/etc/octopus/default/tentacle-default.config"
applicationPath="/home/Octopus/Applications/"
spaceName="#{Octopus.Space.Name}"

sudo apt install --no-install-recommends gnupg curl ca-certificates apt-transport-https && \
curl -sSfL https://apt.octopus.com/public.key | sudo apt-key add - && \
sudo sh -c "echo deb https://apt.octopus.com/ stable main > /etc/apt/sources.list.d/octopus.com.list" && \
sudo apt update && sudo apt install tentacle -y

sudo /opt/octopus/tentacle/Tentacle create-instance --config "$configFilePath"
sudo /opt/octopus/tentacle/Tentacle new-certificate --if-blank
sudo /opt/octopus/tentacle/Tentacle configure --port 10933 --noListen False --reset-trust --app "$applicationPath"
sudo /opt/octopus/tentacle/Tentacle configure --trust $thumbprint
echo "Registering the Tentacle $name as a worker with server $serverUrl in $workerPoolName"
sudo /opt/octopus/tentacle/Tentacle register-worker --server "$serverUrl" --apiKey "$apiKey" --name "$name" --space "$spaceName" --publicHostName "$publicHostName" --workerpool "$workerPoolName"


# Download the Microsoft repository GPG keys
wget -q https://packages.microsoft.com/config/ubuntu/18.04/packages-microsoft-prod.deb

# Register the Microsoft repository GPG keys
sudo dpkg -i packages-microsoft-prod.deb

# Update the list of products
sudo apt-get update

# Enable the "universe" repositories
sudo add-apt-repository universe

# Install PowerShell
sudo apt-get install -y powershell

# Install and start the Tentacle service
sudo /opt/octopus/tentacle/Tentacle service --install --start
'@

Write-Output "Converting cloudInitScript to base64"

# Convert to base 64
$cloudInitScript = [System.Convert]::ToBase64String([system.Text.Encoding]::UTF8.GetBytes($cloudInitScript))

# Set output variable
Set-OctopusVariable -name "CloudInitScript" -value $cloudInitScript
```

With our script converted to a Base64 string, we can pass it to our ARM template

![](octopus-arm-template-custom-data.png)


#### Wait for VM
This step is another Run a Script step that waits for the newly created VM to respond and let us know it's available for use.  Here's the script (Test-NetConnect returns warnings if the destination is unreachable, this is normal as we wait for the VM to be available).

```PS
# Define variables
$publicDNS = "#{Azure.Worker.DNS.Prefix}.#{Azure.Region.Name}.cloudapp.azure.com"

$connectionTest = Test-NetConnection -ComputerName $publicDNS -Port 10933 -WarningAction SilentlyContinue -InformationLevel Quiet

while ($connectionTest -eq $false)
{
  # Give it five seconds
  Start-Sleep -Seconds 5

  # Server not quite ready
  $connectionTest = Test-NetConnection -ComputerName $publicDNS -Port 10933 -WarningAction SilentlyContinue -InformationLevel Quiet
}
```

#### Perform health check
The final step of our Runbook is to perform a health check on the newly created worker.  The **Health Check** built-in step template won't work for us in this as it is designed to work against a Deployment Target and not a Worker.  This step is another Run a Script step that calls the Octopus Deploy API to initiate a health check on a worker

```PS
# Define parameters
$baseUrl = $OctopusParameters['Global.Base.Url']
$apiKey = $OctopusParameters['Global.Api.Key']
$spaceId = $OctopusParameters['Octopus.Space.Id']
$spaceName = $OctopusParameters['Octopus.Space.Name']
$environmentName = $OctopusParameters['Octopus.Environment.Name']

# Get worker pool
$workerPool = (Invoke-RestMethod -Method Get -Uri "$baseUrl/api/$spaceId/workerpools/all" -Headers @{"X-Octopus-ApiKey"="$apiKey"}) | Where-Object {$_.Name -eq "#{PoolName}"}

# Get worker
$worker = (Invoke-RestMethod -Method Get -Uri "$baseUrl/api/$spaceId/workerpools/$($workerPool.Id)/workers" -Headers @{"X-Octopus-ApiKey"="$apiKey"}).Items | Where-Object {$_.Name -eq "$spaceName-$environmentName"}

# Build payload
$jsonPayload = @{
    Name = "Health"
    Description = "Check $spaceName-$environmentName health"
    Arguments = @{
        Timeout = "00:05:00"
        MachineIds = @(
            $worker.Id
        )
    OnlyTestConnection = "false"
    }
    SpaceId = "$spaceId"
}

# Execute health check
Invoke-RestMethod -Method Post -Uri "$baseUrl/api/tasks" -Body ($jsonPayload | ConvertTo-Json -Depth 10) -Headers @{"X-Octopus-ApiKey"="$apiKey"}
```

When done, your Runbook should look something like this,

![](octopus-project-create-vm-runbook.png)

## Executing a PowerShell step template
Now comes the fun part!  In this example, I'm using the [MariaDB - Create Database If Not Exists](https://library.octopus.com/step-templates/2bdfe600-e205-43f9-b174-67ee5d36bf5b/actiontemplate-mariadb-create-database-if-not-exists) step template.  As the name implies, it connects to a MariaDB server and creates a database.  Not only is this step template written completely in PowerShell, it also makes use of a third-party PowerShell module called [SimplySql](https://www.powershellgallery.com/packages/SimplySql/1.6.2).  During execution, if the template detects that SimplySql is not installed, it downloads it to a temporary folder and uses Import-Module so that it's included for the deployment!



As you can see, the step template executes, downloads, and imports the third-party PowerShell module flawlessly!


## Conclusion
A limiting factor in Octopus Deploy adoption amongst those in the world of Linux was the limited selection of templates that could be executed.  With PowerShell core, the selection of templates that can execute on Linux machines is opened exponentially.