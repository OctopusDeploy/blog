---
title: Importing Variables from SSISDB
description: Use PowerShell and the Octopus Deploy API to automate importing variables from SSISB
author: shawn.sesna@octopus.com
bannerImage: importing_variables_from_ssisdb.png
metaImage: importing_variables_from_ssisdb.png
visibility: public
published: 2019-06-28
bannerImage:
metaImage:
tags:
 - SSIS
---

## Introduction

If you've ever used the [Deploy ispac SSIS project from a package](https://library.octopus.com/step-templates/bf005449-60c2-4746-8e07-8ba857f93605/actiontemplate-deploy-ispac-ssis-project-from-a-package) or [Deploy ispac SSIS project from Referenced Package](https://library.octopus.com/step-templates/0c8167e9-49fe-4f2a-a007-df5ef2e63fac/actiontemplate-deploy-ispac-ssis-project-from-referenced-package) step template, then you know that it can extract Project Parameters and Connection Manager information from your SSIS package and create them as Environment variables in SSISDB.  You also know that when doing this, every property of a Connection Manager is created as a separate Environment variable. If you don't have a variable named the same in your Octopus project, you get a message:

OctopusParameters collection is empty or CM.OctoDemoSql.AdventureWorks2017.sa.ConnectUsingManagedIdentity not in the collection

The variable is still created in the SSISDB Environment, however, it defaults to the design time values.  If your SSIS package has large number of Project Paramaters and/or Connection Managers, the list of variables can be quite extensive and, let's be honest, quite tedious to create one-by-one.  

## Automation to the Rescue!

As the original author of Deploy ispac SSIS project from a package, I can tell you that my SSIS developers came pounding on my office door with pitchforks and torches, ranting at how time consuming it was to create all those variables.  To avoid death from the daggers in their eyes, I turned to PowerShell and the Octopus Deploy API to come up with a method to retrieve the variables from the SSISDB Environment and import them into their Octopus Deploy projects.

### The Script

:::warning
The following script is provided for demonstration purposes.
:::
  The script below pulls variables and values from the SSISDB Environment and creates them as project variables in Octopus Deploy!  This saved a TON of time and the devs departed my office, satiated that their demands had been met.

```PS
# Define parameters
param(
    $OctopusServerUrl,
    $APIKey
)

# Define functions
Function Get-EnvironmentVariablesFromSSISDB
{
    # Define parameters
    param(
        $UseIntegratedAuthentication,
        $SqlUserName,
        $SqlPassword,
        $CatalogName,
        $FolderName,
        $EnvironmentName,
        $SqlServerName
    )

    # Import needed assemblies
    [Reflection.Assembly]::LoadWithPartialName("Microsoft.SqlServer.Management.IntegrationServices") | Out-Null # Out-Null supresses a message that would normally be displayed saying it loaded out of GAC

    # Create a connection to the server
    $sqlConnectionString = "Data Source=$SqlServerName;Initial Catalog=master;"
    
    # Check authentication
    if ($UseIntegratedAuthentication)
    {
        # Add integrated
        $sqlConnectionString += "Integrated Security=SSPI;"
    }
    else 
    {
        # ass username password
        $sqlConnectionString += "User ID=$SqlUserName; Password=$SqlPassword"    
    }
    
    $sqlConnection = New-Object System.Data.SqlClient.SqlConnection $sqlConnectionString

    # create integration services object
    $integrationServices = New-Object "$ISNamespace.IntegrationServices" $sqlConnection

    try
    {
        # get catalog reference
        $Catalog = Get-Catalog -CatalogName $CataLogName -IntegrationServices $integrationServices
        $Folder = Get-Folder -FolderName $FolderName -Catalog $Catalog
        $Environment = Get-Environment -Folder $Folder -EnvironmentName $EnvironmentName

        # return environment variables
        return $Environment.Variables 
    }
    finally
    {
        # close connection
        $sqlConnection.Close()
    }
}

Function Get-Folder
{
    # parameters
    Param($FolderName, $Catalog)

    # try to get reference to folder
    $Folder = $Catalog.Folders[$FolderName]

    # check to see if $Folder has a value
    if(!$Folder)
    {
        Write-Error "Folder not found."
        throw
    }
    
    # return the folde reference
    return $Folder
}

Function Get-Environment
{
    # define parameters
    Param($Folder, $EnvironmentName)

    # get reference to Environment
    $Environment = $Folder.Environments[$EnvironmentName]

    # check to see if it's a null reference
    if(!$Environment)
    {
        Write-Error "Environment not found."
        throw
    }

    # return the environment
    return $Environment
}

Function Get-Catalog
{
    # define parameters
    Param ($CatalogName, $IntegrationServices)

    # define working varaibles
    $Catalog = $null

    # check to see if there are any catalogs
    if($integrationServices.Catalogs.Count -gt 0 -and $integrationServices.Catalogs[$CatalogName])
    {
        # get reference to catalog
        $Catalog = $integrationServices.Catalogs[$CatalogName]
    }
    else
    {
        Write-Error  "Catalog $CataLogName does not exist or the Tentacle account does not have access to it."

        # throw error
        throw 
    }

    # return the catalog
    return $Catalog
}

Function Get-OctopusProject
{
    # Define parameters
    param(
        $OctopusServerUrl,
        $ApiKey,
        $ProjectName
    )

    # Call API to get all projects, then filter on name
    $octopusProject = Invoke-RestMethod -Method "get" -Uri "$OctopusServerUrl/api/projects/all" -Headers @{"X-Octopus-ApiKey"="$ApiKey"}

    # return the specific project
    return ($octopusProject | Where-Object {$_.Name -eq $ProjectName})
}

Function Get-OctopusProjectVariables
{
    # Define parameters
    param(
        $OctopusDeployProject,
        $OctopusServerUrl,
        $ApiKey
    )

    # Get reference to the variable list
    return (Invoke-RestMethod -Method "get" -Uri "$OctopusServerUrl/api/variables/$($OctopusDeployProject.VariableSetId)" -Headers @{"X-Octopus-ApiKey"="$ApiKey"})
}

Function Update-ProjectVariables
{
    param(
        $OctopusServerUrl,
        $ProjectVariables,
        $ApiKey
    )

    # Convert the object into JSON
    $jsonBody = $ProjectVariables | ConvertTo-Json -Depth 5

    # Call the API to update
    Invoke-RestMethod -Method "put" -Uri "$OctopusServerUrl/api/variables/$($ProjectVariables.Id)" -Body $jsonBody -Headers @{"X-Octopus-ApiKey"="$ApiKey"}
}

try
{
    # Store the IntegrationServices Assembly namespace to avoid typing it every time
    $ISNamespace = "Microsoft.SqlServer.Management.IntegrationServices"
    $CataLogName = "SSISDB"

    # Get reference to project
    $octopusProject = Get-OctopusProject -OctopusServerUrl "<Your URL Here>" -ApiKey "<Your API key here>" -ProjectName "<Octopus Deploy project name>"

    # Get list of existing variables
    $octopusProjectVariables = Get-OctopusProjectVariables -OctopusDeployProject $octopusProject -OctopusServerUrl "<Your URL Here>" -ApiKey "<Your API key here>"

    # Get list of SSIS project variables
    $ssisEnvironmentVariables = Get-EnvironmentVariablesFromSSISDB -UseIntegratedAuthentication $false -SqlServerName "<Sql server name>" -SqlUserName "<sql account user name>" -SqlPassword "<sql account password>" -CatalogName "SSISDB" -FolderName "<SSISDB folder name>" -EnvironmentName "<SSISDB environment name>"

    # Loop through the ssis variable set
    foreach ($variable in $ssisEnvironmentVariables)
    {
        # Check to see if variable already exists in Octopus project variables
        if ($null -eq ($octopusProjectVariables.Variables | Where-Object {$_.Name -eq $variable.Name}))
        {
            # Display message
            Write-Output "Adding $($variable.Name) to Octopus Deploy project $($octopusProject.Name)"
            
            # Create new variable hash table
            $newVariable = @{
                #Id = "$(New-Guid)"
                Name = "$($variable.Name)"
                Value = "$($variable.Value)"
                Description = $null
                Scope = @{}
                IsEditable = $(if ($variable.Sensitive) { $false} else {$true})
                Prompt = $null
                Type = "String"
                IsSensitive = $(if ($variable.Sensitive) { $true} else {$false})
            }

            # Add variable
            $octopusProjectVariables.Variables += $newVariable
        }
    }

    # Update the project 
    Update-ProjectVariables -ProjectVariables $octopusProjectVariables -ApiKey $APIKey -OctopusServerUrl $OctopusServerUrl
}
catch
{
    Write-Error $_.Exception.Message

    throw
}
```

## Summary
In this post I showed you a quick way to populate the Octopus Deploy project variables by connecting to SSISDB and copying the Environment variables.  This solution does require the SSIS package be deployed at least once so that the Environment variables get populated in SSISDB.  Though this example was specific to SSISDB, the general approach of using the API to programatically add variables to an Octopus Deploy project can be used with a wide variety of sources.