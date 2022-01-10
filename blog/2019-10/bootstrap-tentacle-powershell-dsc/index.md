---
title: Bootstrap Tentacle installation with Powershell DSC
description: Install and configure Tentacles using the power of Powershell Desired State Configuration (DSC)
author: shawn.sesna@octopus.com
visibility: public
published: 2019-10-01
bannerImage: bootstrapping_tentacle_installation_dsc.png
bannerImageAlt: Tentacles rising from the ocean to install software on a server
metaImage: bootstrapping_tentacle_installation_dsc.png
tags:
 - DevOps
 - PowerShell
---

![Tentacles rising from the ocean to install software on a server](bootstrapping_tentacle_installation_dsc.png)

Manually installing Tentacles on deployment targets is fairly painless when you have a small number of machines, with just a couple of clicks you’re done. However, it’s not uncommon for the number of machines to increase exponentially as adoption of Octopus Deploy becomes more prevalent, especially in large organizations.  In more advanced implementations, targets are sometimes created dynamically, and manually installing Tentacles isn’t feasible.  This is where Infrastructure as Code (IaC) comes in handy.

Infrastructure as Code is an awesome and powerful concept.  It gives you the ability to programmatically define how your infrastructure should be set up, which leads to consistency and predictability for application deployments. One method of IaC is PowerShell Desired State Configuration (DSC).  Not only can DSC configure a machine, it also gives us the added benefit of monitoring the machine for drift and using Octopus to automatically return the machine to the desired state.  In this post, I’ll walk you through using the OctopusDSC module to install and configure a Tentacle machine.

## Install the NuGet package provider

There is a drawback to using DSC, any external module you use needs to be present on the machine before the DSC script runs.  This means you have to separate the installation of the external module from the DSC script itself.

In order to download external modules, we first need to install the NuGet package provider.  Depending on your server configuration, it may be necessary to include TLS 1.2:

```PS
# Include use of TLS 1.2
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor [System.Net.SecurityProtocolType]::Tls12


# check to see if Nuget is installed
if($null -eq (Get-PackageProvider | Where-Object {$_.Name -eq "NuGet"}))
{
    # install the nuget package provider
    Install-PackageProvider -Name NuGet -Force
}
```

## Download the OctopusDSC module

Now that we have the NuGet package provider installed, we can contact the PowerShell gallery to download and install the modules we need:


```PS
if ($null -eq (Get-Package | Where-Object {$_.Name -eq "OctopusDSC"}))
{
    # download specified module
    Install-Module -Name "OctopusDSC" -Force
}
```

## Setting up the DSC resource

Now that we’ve taken care of the pre-requisite components, it’s time to set up our OctopusDSC cTentacleAgent resource. There are a bunch of parameters we can pass to this resource outlined [here](https://github.com/OctopusDeploy/OctopusDSC/blob/master/README-cTentacleAgent.md).  For this example, we’ll configure a Listening Tentacle as a deployment target.  If we wanted this to be a worker instead, we’d empty out the Role and Environment arrays and define the entries we want in the WorkerPools array:

```PS

# configure Ocotpus Deploy
Configuration OctopusSetup
{
    # define parameters
    Param([string]$OctopusAPIKey,
        [string[]]$OctopusRoles,
        [string[]]$OctopusEnvironments,
        [string]$OctopusServerUrl,
        [string]$DefaultApplicationDirectory,
        [string]$TentacleHomeDirectory,
        [string[]]$WorkerPools,
        [PSCredential]$TentacleServiceCredential,
        [string]$TentacleInstanceName = "Default",
        [ValidateSet("Listen", "Poll")]
        [string]$CommunicationMode = "Listen",
        [ValidateSet("PublicIp", "FQDN", "ComputerName", "Custom")]
        [string]$PublicHostNameConfiguration = "ComputerName"
    )

    # import necessary resources
    Import-DscResource -Module OctopusDSC
    Import-DscResource -ModuleName PSDesiredStateConfiguration

    # create localhost configuration node
    Node 'localhost'
    {
        cTentacleAgent OctopusTentacle
        {
            Name = $TentacleInstanceName
            DisplayName = $env:COMPUTERNAME
            Ensure = "Present"
            State = "Started"
            ApiKey = $OctopusAPIKey
            OctopusServerUrl = $OctopusServerUrl
            Environments = $OctopusEnvironments
            Roles = $OctopusRoles
            CommunicationMode = $CommunicationMode
            DefaultApplicationDirectory = $DefaultApplicationDirectory
            TentacleHomeDirectory = $TentacleHomeDirectory
            WorkerPools = $WorkerPools
            PublicHostNameConfiguration = $PublicHostNameConfiguration
            TentacleServiceCredential = $TentacleServiceCredential
        }
    }
}


# Example roles
$OctopusRoles = @("ExampleRole")

# Example Environments
$OctopusEnvironments = @("Development")

# Example worker pools
$WorkerPools = @()

# Set the APIKey
$OctopusAPIKey = "API-XXXXXXXXXXXXXXXXXXXXXX"

# Set the Octopus Server URL
$OctopusServerUrl = "https://<YourOctoupsServer>"

# Set directories
$DefaultApplicationDirectory = "c:\octopus"
$TentacleHomeDirectory = "c:\octopus\tentaclehome"

# run configuration
OctopusSetup -OctopusAPIKey $OctopusAPIKey -OctopusRoles $OctopusRoles -OctopusEnvironments $OctopusEnvironments -OctopusServerUrl $OctopusServerUrl -OutputPath "c:\dsc" -DefaultApplicationDirectory $DefaultApplicationDirectory -TentacleHomeDirectory $TentacleHomeDirectory -WorkerPools $WorkerPools -TentacleServiceCredential $serviceCredential

# start the configuration
Start-DscConfiguration -Path "c:\dsc" -Verbose -Wait

```

## Putting it all together

So far, we’ve set up individual components to demonstrate what we’re trying to achieve.  Now, let’s take all of this and put it into a single script.

:::hint
Even though we will run the script blocks on a remote computer, the PowerShell is evaluated on our local machine first.  If you recall, the DSC resource must exist before it’ll work, which means we need to have the OctopusDSC resource on the machine we’re running the script from.  Annoying, I know, but that’s how it works.
:::

```PS
# Get credentials that have rights to the server
$elevatedCredentials = Get-Credential

# Create remote session
$remoteSession = New-PSSession -ComputerName <ComputerName> -Credential $elevatedCredentials

# Download and install the Nuget package provider
Invoke-Command -Session $remoteSession -ScriptBlock {
    # Include use of TLS 1.2
    [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor [System.Net.SecurityProtocolType]::Tls12

    # check to see if Nuget is installed
    if((Get-PackageProvider | Where-Object {$_.Name -eq "NuGet"}) -eq $null)
    {
        # install the nuget package provider
        Install-PackageProvider -Name NuGet -Force
    }
}

# Download and install the OctopusDSC module
Invoke-Command -Session $remoteSession -ScriptBlock{
    if ($null -eq (Get-Package | Where-Object {$_.Name -eq "OctopusDSC"}))
    {
        # download specified module
        Install-Module -Name "OctopusDSC" -Force
    }
}

# Run the DSC configuration
Invoke-Command -Session $remoteSession -ScriptBlock{

    # configure Ocotpus Deploy
    Configuration OctopusSetup
    {
	    # define parameters
	    Param([string]$OctopusAPIKey,
		    [string[]]$OctopusRoles,
		    [string[]]$OctopusEnvironments,
		    [string]$OctopusServerUrl,
		    [string]$DefaultApplicationDirectory,
		    [string]$TentacleHomeDirectory,
		    [string[]]$WorkerPools,
		    [PSCredential]$TentacleServiceCredential,
            [string]$TentacleInstanceName = "Default",
            [ValidateSet("Listen", "Poll")]
            [string]$CommunicationMode = "Listen",
            [ValidateSet("PublicIp", "FQDN", "ComputerName", "Custom")]
            [string]$PublicHostNameConfiguration = "ComputerName"
	    )

	    # import necessary resources
	    Import-DscResource -Module OctopusDSC
	    Import-DscResource -ModuleName PSDesiredStateConfiguration

	    # create localhost configuration node
	    Node 'localhost'
	    {
            cTentacleAgent OctopusTentacle
		    {
			    Name = $TentacleInstanceName
                DisplayName = $env:COMPUTERNAME
                Ensure = "Present"
			    State = "Started"
			    ApiKey = $OctopusAPIKey
			    OctopusServerUrl = $OctopusServerUrl
			    Environments = $OctopusEnvironments
			    Roles = $OctopusRoles
                CommunicationMode = $CommunicationMode
                DefaultApplicationDirectory = $DefaultApplicationDirectory
                TentacleHomeDirectory = $TentacleHomeDirectory
			    WorkerPools = $WorkerPools
                PublicHostNameConfiguration = $PublicHostNameConfiguration
                TentacleServiceCredential = $TentacleServiceCredential
		    }
	    }
    }

    # Example roles
    $OctopusRoles = @("ExampleRole")

    # Example Environments
    $OctopusEnvironments = @("Development")

    # Example worker pools
    $WorkerPools = @()

    # Set the APIKey
    $OctopusAPIKey = "API-XXXXXXXXXXXXXXXXXXXXXX"

    # Set the Octopus Server URL
    $OctopusServerUrl = "https://YourOctopusServer"

    # Set directories
    $DefaultApplicationDirectory = "c:\octopus"
    $TentacleHomeDirectory = "c:\octopus\tentaclehome"

    # run configuration
    OctopusSetup -OctopusAPIKey $OctopusAPIKey -OctopusRoles $OctopusRoles -OctopusEnvironments $OctopusEnvironments -OctopusServerUrl $OctopusServerUrl -OutputPath "c:\dsc" -DefaultApplicationDirectory $DefaultApplicationDirectory -TentacleHomeDirectory $TentacleHomeDirectory -WorkerPools $WorkerPools -TentacleServiceCredential $serviceCredential

    # start the configuration
    Start-DscConfiguration -Path "c:\dsc" -Verbose -Wait
}

```

Using this script, we can install and configure a Tentacle machine without having to RDP to it. This is especially useful when the scaling infrastructure dynamically.

## Testing for drift and Octopus machine policies

As the name implies, DSC is what we want the desired state to be.  For example, the above configuration configured the Tentacle to have the role (and only the role) of ExampleRole.  If someone added an additional role to this Tentacle, it would no longer be in the desired state. We can determine this by running:

```PS
(Test-DscConfiguration -Detailed).ResourcesNotInDesiredState
```
Using the Machine Policy feature of Octopus Deploy, we can create a new policy that will automatically correct drift if it’s detected.

To accomplish this, navigate to the Infrastructure tab and click on Machine Policies.  Once there, click the Add Machine Policy button:

![](add-new-machine-policy.png)

Select the Use Custom Script radio button and paste the following:

![](machine-policy-powershell.png)

```PS
$tentacleConfiguration = (Test-Configuration -Detailed)

if ($null -ne $tentacleConfiguration.ResourcesNotInDesiredState)
{
    # Display what resources are not in desired state
    foreach ($resource in $tentacleConfiguration.ResourcesNotInDesiredState)
    {
        Write-Warning "Resource $($resource.ResourceId) is not in desired state!"
    }

    Write-Output "Running last DSC configuration to correct drift..."
    Start-DscConfiguration -Path "c:\dsc"
}
```

Now, when drift is detected it will automatically run the last DSC configuration and put the machine back in the desired state. In the example above, if somebody added an additional role to a Tentacle, this policy will detect the drift and remove the added role.  If the role is something you need, you can add it to the `$OctopusRoles` variable in our original script and then run the new configuration to set the new desired state.

## Conclusion

In this post, I demonstrated how to install and configure a Tentacle machine with one simple script as well as a method for detecting and correcting drift.  Using a method like DSC helps to ensure that all of your installations are done consistently.
