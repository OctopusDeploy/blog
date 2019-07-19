---
title: Verify appSettings or variable replacement
description: How to programatically verify that your AppSettings have a matching Octopus Deploy variable defined and/or verify all variable replacement in files succeeded.
author: shawn.sesna@octopus.com
visibility: public
bannerImage: 
metaImage: 
published: 2019-07-20
tags:
 - Octopus
---

## Introduction
There is nothing worse than having uncertainty in a deployment process, heck that's one of the reasons Octopus Deploy exists!  To combat uncertainty, Octopus Deploy offers both consistency and reliability in that the process for deployment will always be the same from environment to environment.  However, there is one thing that can change when deploying to the next environment, Variables.  Since variables are able to be changed from one environment to the next, we must be diligent that we have spelled them the same and/or scoped them accordingly.  This post will provide some examples of how to detect missing or unchanged variables in a deployment step.

## .config files
It is often the case that values within a .config file need to be different based on the environment being deployed to such as connection strings or app settings.  It is also often the case that these .config files are maintanied by the developers who may not have access to Octopus Deploy.  In those cases, the communication to have the variable added to the Octopus Deploy project may get lost in the shuffle and may go undetected until the application is deployed to Production where the developer known value doesn't work.    

### Check all app settings
Any step template that offers the feature of Pre-Deployment scripts will work with this solution.  For this example, we're going to be using the Deploy to IIS step template.  Once you've added a Deploy to IIS step to your process, click on the Configure Features button

![](configure-features.png)

Next, enable the Custom Deployment Scripts feature

![](enable-custom-scripts.png)

Expand the Custom Deployment Scripts section of the step template

![](expand-section.png)

Enter the following code into the Pre-deployment script window

```PS
# Get the config file
$configFile = Get-ChildItem -Path $OctopusParameters['Octopus.Action.Package.InstallationDirectoryPath'] | Where-Object {$_.Name -eq "web.config"}

# Create FileMap object
$fileMap = New-Object System.Configuration.ExeConfigurationFileMap

# Set location of exe config file
$fileMap.ExeConfigFilename = $configFile

# Create Configuration Manager object
$configManager = [System.Configuration.ConfigurationManager]::OpenMappedExeConfiguration($fileMap, [System.Configuration.ConfigurationUserLevel]::None)

# Iterate through appSettings collection
foreach($appSetting in $configManager.AppSettings.Settings)
{
	# Check to see if key is present
    if (!$OctopusParameters.ContainsKey($appSetting.Key))
    {
    	# Fail the deployment
        throw "Octopus Parameter collection does not contain a value for $($appSetting.Key)"
    }
}
```

And that's it!  Now, before the step deploys, we check to make sure that all app settings keys are present in the Octopus Parameters collection and fail the deployment if it's not found!

### What about static app settings?
Fantastic question!  The above example really only works when you have all of your app settings defined as Octopus Deploy variables.  There are plenty of cases where you would have a static list of app settings that don't ever change and doesn't make sense to make them variables.  For this, we can still use the same approach, but tweak it to exclude a list of key values.

```PS
# Define array of app settings to ignore
$settingsToIgnore = @("somesetting", "someothersetting")

# Get the config file
$configFile = Get-ChildItem -Path $OctopusParameters['Octopus.Action.Package.InstallationDirectoryPath'] | Where-Object {$_.Name -eq "web.config"}

# Create FileMap object
$fileMap = New-Object System.Configuration.ExeConfigurationFileMap

# Set location of exe config file
$fileMap.ExeConfigFilename = $configFile

# Create Configuration Manager object
$configManager = [System.Configuration.ConfigurationManager]::OpenMappedExeConfiguration($fileMap, [System.Configuration.ConfigurationUserLevel]::None)

# Iterate through appSettings collection
foreach($appSetting in $configManager.AppSettings.Settings)
{
	# Check to see if key is present
    if ((!$OctopusParameters.ContainsKey($appSetting.Key)) -and ($settingsToIgnore -notcontains $appSetting.Key))
    {
    	# Fail the deployment
        throw "Octopus Parameter collection does not contain a value for $($appSetting.Key)"
    }
    #Write-Output "Setting is $($appSetting.Key)"
}
```
This version of the code will ignore any setting defined in the $settingsToIgnore array.  This way you ignore everything you don't care about, but catch anything new that might have been added.

## Substitute Variables in Files
Another common pain point when dealing with variables is the Substitute Variables in Files feature.  When Using the Sustitute Variables in Files feature, Octopus Deploy will only replace variable placeholders in files if it has a matching variable in the collection. If there wasn't a matching variable in the collection for a placeholder in a file, Octopus Deploy does not warn you.  As you can imagine, this can be quite problematic. 

 Are we able to preempitvely fail a step if a file still has the #{} placeholder in it?  YES!  However, the variable relacement occurs during the Deployment phase, so we're unable to use the Pre-deployment component in this case.  The good news is that the Deployment script occurs just after it so we're still able to fail the deployment before the Deploy to IIS step swaps to the newly deployed folder (if you're using Custom Installation Directory, this may not work as expected).  Just like before, we'll expand the Custom Deployment Scripts section of the Deploy to IIS step and paste the following code in the Deployment script window

```PS
# Get list of files that were specified for substitution
$fileList = $OctopusParameters['Octopus.Action.SubstituteInFiles.TargetFiles']

# Get base install path
$basePath = $OctopusParameters['Octopus.Action.Package.InstallationDirectoryPath']

# Ensure basePath ends with a \
if (!$basePath.EndsWith("\"))
{
	# Add ending slash
    $basePath += "\"
}

# Loop through list of files that were marked for substitution
foreach ($file in $fileList)
{
	# Check to make sure file exists
    if ((Test-Path -Path "$($basePath + $file)") -eq $true)
    {
    	# Read file
        $stringData = Get-Content -Path "$($basePath + $file)"
        
        # Check for token
        if ($stringData.IndexOf("#{") -gt -1)
        {
        	# Something wasn't transformed
            throw "$file still contains #{} syntax."
        }
    }
    else
    {
    	# Display message
        Write-Output "Unable to find file $file."
    }
}
```

And there you have it!  You're now able to proactively verify that your files have the values!

## Conclusion
Scoping and misspelling will always plague the variables component of any automated deployment system.  The examples provided within this post should help mitigate commonly encountered issues.

Happy deployments!