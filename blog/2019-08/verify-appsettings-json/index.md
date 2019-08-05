---
title: Verify App Settings Part Deux
description: How to programmatically verify that your App Settings have a matching Octopus Deploy variable defined and/or verify environment configs have the same JSON structure.
author: shawn.sesna@octopus.com
visibility: public
bannerImage: blogimage-verifyvariables.png
metaImage: blogimage-verifyvariables.png
published: 
tags:
 - Octopus
---

## Introduction
In my [last post](https://octopus.com/blog/verify-appsettings-or-variable-replacement), I showed you how to verify that all of your App Settings in a web.config file had corresponding Octopus project variables as well as making sure all of your files that were configured for variable replacement didn't have any leftover placeholders.  This post will focus on verifying App Settings in .NET Core appSettings.json.

## Comparing environmental files
It is not uncommon to have environmental App Settings files for a .NET core application such as appSettings.Developement.json.  The caveat to using this approach is that if something is added to the appSettings.Deveopment.json file, it needs to be added to the appSettings.json file as well.  To address this issue, we can create a PowerShell Function with a recursive call to traverse the json file keys then compare the entries to the environmental files.  Similar to the last post, we can define any settings that we might want to ignore using the `$settingsToIgnore` array.

```PS
Function Get-AppSettings
{
    # Define parameters
    param($jsonObject,
        $parentNamespace,
        $settingsToIgnore)

    $namespace = ""
    $tempArray = @()


    if (![string]::IsNullOrEmpty($jsonObject) -and ($jsonObject.GetType().Name -eq "PSCustomObject"))
    {
        
        # Get number of properties
        $properties = $jsonObject | Get-Member -MemberType Properties | Select-Object -ExpandProperty Name

        # Check to see if it has properties
        if ($properties.Length -gt 0)
        {
            # Loop through returned properties
            foreach($property in $properties)
            {
                # Make sure we not supposed to ignore it
                if ($null -eq $parentNamespace)
                {
                    # Assign the property value
                    $namespace = $property
                }
                else
                {
                    $namespace = "$parentNamespace.$property"
                }

                # Make sure we're not supposed to ignore it
                if ($settingsToIgnore -notcontains $namespace)
                {
                    # Add the namespace to the array
                    $tempArray += $namespace

                    # Add the returned array from the recursive call
                    $tempArray += Get-AppSettings -jsonObject ($jsonObject.$property) -parentNamespace $namespace -settingsToIgnore $settingsToIgnore
                }
            }
        }
        else
        {
            # Add the value to the array
            if ($null -eq $parentNamespace)
            {
                $namespace = $property
            }
            else
            {
                $namespace = "$parentNamespace.$property"
            }

            $tempArray += $namespace
        }
    }

    return $tempArray
}

# Load the json file
$appSettingsFile = Get-Content -Path (Get-ChildItem -Path $OctopusParameters['Octopus.Action.Package.InstallationDirectoryPath'] | Where-Object {$_.Name -eq "appsettigs.config"}).FullName | ConvertFrom-Json

# Define working variable
$appSettings = @()
$settingsToIgnore = @("octofront.azure")

# Get all the json keys
$appSettings = Get-AppSettings -jsonObject $appSettingsFile -parentNamespace $null -settingsToIgnore $settingsToIgnore

# Get all files that are appsettings.something.json
$otherAppSettingsFiles = Get-ChildItem -Path  $OctopusParameters['Octopus.Action.Package.InstallationDirectoryPath']  | Where-Object {$_.Name -match "^appSettings.*..json"}

# Loop through other files
foreach ($otherFile in $otherAppSettingsFiles)
{
    # Convert the json to PowerShell objects
    $otherSettingsFile = Get-Content -Path $otherFile.FullName  | ConvertFrom-Json

    # Get the json keys
    $otherSettings = Get-AppSettings -jsonObject $otherSettingsFile -parentNamespace $null -settingsToIgnore $settingsToIgnore

    # Compare the properties to see if they are identical
    $results = Compare-Object -ReferenceObject $appSettings -DifferenceObject $otherSettings 

    # Check to see if something was returned
    if ($null -ne $results)
    {
        # Extract results
        $resultsError = $results | Out-String

        throw "Differences found in $($otherFile.FullName)`n $resultsError"
    }
}
```

Now, we are able to detect of if we accidently forgot to add a setting to the main appsettings.json file and prevent the deployment from executing!

## Ensuring App Settings have Octopus Parameters

Using the function `Get-AppSettings` previously defined as well as the `$settingsToIgnore` array, we can use the following code to ensure that the settings defined in App Settings have corresponding Octopus Deploy Variables.

```PS
# Load the json file
$appSettingsFile = Get-Content -Path (Get-ChildItem -Path $OctopusParameters['Octopus.Action.Package.InstallationDirectoryPath'] | Where-Object {$_.Name -eq "appsettigs.config"}).FullName | ConvertFrom-Json

# Define working variable
$appSettings = @()
$settingsToIgnore = @("octofront.azure")

# Get all the json keys
$appSettings = Get-AppSettings -jsonObject $appSettingsFile -parentNamespace $null -settingsToIgnore $settingsToIgnore

foreach ($appSetting in $appSettings)
{
	# Check to see if key is present
    if (!$OctopusParameters.ContainsKey($appSetting.Replace(".", ":"))) # Variables have : delimeters for nested json keys
    {
    	# Fail the deployment
        throw "Octopus Parameter collection does not contain a value for $($appSetting)"
    }    
}
```