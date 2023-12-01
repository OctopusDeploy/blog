---
title: How to bulk update the execution container image
description: Learn how to use an API script to update the image used for Execution Containers in deployment processes and runbooks.
author: shawn.sesna@octopus.com
visibility: private
published: 3020-01-01
metaImage: to-be-added-by-marketing
bannerImage: to-be-added-by-marketing
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: <!-- see https://github.com/OctopusDeploy/blog/blob/master/tags.txt for a comprehensive list of tags -->
 - DevOps
 - Product
 - Engineering
---


The [Execution Containers](https://octopus.com/docs/projects/steps/execution-containers-for-workers) feature in Octopus Deploy makes it easy to ensure you have the necessary tooling when running your steps.  As with all things that use containers, you need to specify either the specific tag for the version you want to use or always accept the latest version.  For various reasons, it may be necessary to update the image and or tag used with Execution containers which can be quite time-consuming if you have a lot of processes or runbooks that make use of them.  In this post, I'll demonstrate how to use PowerShell and the Octopus API to programmatically update the Execution Container Image.


## Body


If you’re familiar with the [Samples](https://samples.octopus.app) instance, you'll know that the projects make heavy use of the Execution Containers features.  As the maintainer of Samples, it is my job to make sure that the examples continue to function and deploy properly.  Many of the example projects used the [octopusdeploy/worker-tools](https://hub.docker.com/r/octopusdeploy/worker-tools) image as it contains commonly used tooling to perform a multitude of tasks.  However, this image does not make use of the `latest` tag, so the tag version must be specified in every step that makes use of it.  The example projects on Samples were developed when the standard Ubuntu image was version 18.04, which is now deprecated and the `worker-tools` image was tagged thusly.  Going through each project and updating the referenced container one by one was less than appealing, Octopus API to the rescue!


### The script


Knowing this would be a continuing issue to deal with, I wrote a PowerShell script that uses the Octopus API to iterate through all Spaces, Projects, and Runbooks and update them to use the image that I've specified.  


:::Information
The following script is provided as a demonstration of how to perform bulk operations on Deployment and Runbook steps.  Please review carefully if you intend to run this on your own instance.
:::


#### Variables


The script requires a few variables for execution:
- `$octopusUrl`: The Url to your Octopus instance.
- `$octopusAPIKey`: API key with sufficient permissions to update the deployment and runbook processes.
- `$linuxWorkerToolsImage`: The Linux image to update to (e.g. octopusdeploy/worker-tools:ubuntu.22.04)
- `$windowsWorkerToolsImage`: The Windows image to update to (e.g. octopusdeploy/worker-tools:windows.ltsc2019)


#### The script
The script below will iterate through all Spaces, Deployment Processes, and Runbook Processes looking for an Execution Container is used.  If found, it will check to see if the name of the image contains `octopusdeploy/worker-tools` and update the image to the one specified in the `$linuxWorkerToolsImage` or `$windowsWorkerToolsImage`.  


**Note**: Version controlled projects and Runbooks that have never been published are skipped.  I made an assumption that Runbooks that have never been published are that way on purpose.


```powershell
$ErrorActionPreference = "Stop";


function Get-OctopusItems
{
    # Define parameters
    param(
        $OctopusUri,
        $ApiKey,
        $SkipCount = 0
    )
   
    # Define working variables
    $items = @()
    $skipQueryString = ""
    $headers = @{"X-Octopus-ApiKey"="$ApiKey"}


    # Check to see if there there is already a querystring
    if ($octopusUri.Contains("?"))
    {
        $skipQueryString = "&skip="
    }
    else
    {
        $skipQueryString = "?skip="
    }


    $skipQueryString += $SkipCount
   
    # Get intial set
    Write-Host "Calling $OctopusUri$skipQueryString"
    $resultSet = Invoke-RestMethod -Uri "$($OctopusUri)$skipQueryString" -Method GET -Headers $headers


    # Check to see if it returned an item collection
    if ($null -ne $resultSet.Items)
    {
        # Store call results
        $items += $resultSet.Items
   
        # Check to see if resultset is bigger than page amount
        if (($resultSet.Items.Count -gt 0) -and ($resultSet.Items.Count -eq $resultSet.ItemsPerPage))
        {
            # Increment skip count
            $SkipCount += $resultSet.ItemsPerPage


            # Recurse
            $items += Get-OctopusItems -OctopusUri $OctopusUri -ApiKey $ApiKey -SkipCount $SkipCount
        }
    }
    else
    {
        return $resultSet
    }
   


    # Return results
    return $items
}


# Define working variables
$octopusURL = "https://YourUrl"
$octopusAPIKey = "API-YourApiKey"
$linuxWorkerToolsImage = "octopusdeploy/worker-tools:ubuntu.22.04"
$windowsWorkerToolsImage = "octopusdeploy/worker-tools:windows.ltsc2019"
$header = @{ "X-Octopus-ApiKey" = $octopusAPIKey }


# Get space
$spaces = Get-OctopusItems -OctopusUri "$octopusURL/api/spaces" -ApiKey $octopusAPIKey


foreach ($space in $spaces) {
    Write-Host "Working on space $($space.name)"


    Write-Host "Getting project list ..."                                                                              
    $projects = Get-OctopusItems -OctopusUri "$octopusURL/api/$($space.Id)/projects" -ApiKey $octopusAPIKey


    # Loop through projects
    foreach ($project in $projects)
    {
        # Check to see if project is version controlled
        if ($project.IsVersionControlled -eq $true)
        {
            Write-Host "Project is version controlled, skipping ..."
            continue
        }
       
        # Get project deployment process
        Write-Host "Getting deployment process for $($project.Name)..."
        $deploymentProcess = Get-OctopusItems -OctopusUri "$octopusURL/api/$($space.Id)/deploymentProcesses/$($project.DeploymentProcessId)" -ApiKey $octopusAPIKey
        $processUpdated = $false


        # Analyze steps
        foreach ($step in $deploymentProcess.Steps)
        {
            # Analyze action
            foreach ($action in $step.Actions)
            {
                if (![string]::IsNullOrWhiteSpace($action.Container.Image))
                {
                    # Check to see if the image contains our name
                    if ($action.Container.Image.Contains("octopusdeploy/worker-tools"))
                    {
                        # Determine architecture
                        if ($action.Container.Image.Contains("ubuntu") -and ($action.Container.Image -ne $linuxWorkerToolsImage))
                        {
                            # Update with specified image
                            Write-Host "Updating step $($step.Name) to use $linuxWorkerToolsImage ..."
                            $action.Container.Image = $linuxWorkerToolsImage
                            $processUpdated = $true
                        }


                        if ($action.Container.Image.Contains("windows")-and ($action.Container.Image -ne $windowsWorkerToolsImage))
                        {
                            # Update with specified image
                            Write-Host "Updating step $($step.Name) to use $windowsWorkerToolsImage ..."
                            $action.Container.Image = $windowsWorkerToolsImage
                            $processUpdated = $true
                        }
                    }
                    else
                    {
                        Write-Host "Specified container image does not match 'octopusdeploy/worker-tools', skipping ..."
                    }
                }
            }
        }


        # Update the deployment process
        Write-Host "Updating deployment process for $($project.Name) ..."
        Invoke-RestMethod -Method Put -Uri "$octopusURL/api/$($space.Id)/deploymentProcesses/$($project.DeploymentProcessId)" -Body ($deploymentProcess | ConvertTo-Json -Depth 10) -Headers $header


        # Get runbooks for project
        $runbooks = Get-OctopusItems -OctopusUri "$OctopusURL/api/$($space.Id)/projects/$($project.Id)/runbooks" -ApiKey $octopusAPIKey
       
        foreach ($runbook in $runbooks)
        {
            if ($null -ne $runbook.Id)
            {
               
                if ($null -ne $runbook.PublishedRunbookSnapshotId)
                {
                    # Get published snapshot process
                    $publishedRunbookSnapshot = Get-OctopusItems -OctopusUri "$octopusURL/api/$($space.Id)/runbooksnapshots/$($runbook.PublishedRunbookSnapshotId)" -ApiKey $octopusAPIKey
                    $processUpdated = $false
                    $runbookProcess = Get-OctopusItems -OctopusUri "$octopusURL/api/$($space.Id)/runbookprocesses/$($publishedRunbookSnapshot.FrozenRunbookProcessId)" -ApiKey $octopusAPIKey
                    $currentRunbookProces = Get-OctopusItems -OctopusUri "$octopusURL/api/$($space.Id)/runbookprocesses/$($runbook.RunbookProcessId)" -ApiKey $octopusAPIKey


                    # Analyze steps
                    foreach ($step in $runbookProcess.Steps)
                    {
                        # Analyze action
                        foreach ($action in $step.Actions)
                        {
                            if (![string]::IsNullOrWhiteSpace($action.Container.Image))
                            {
                                # Check to see if the image contains our name
                                if ($action.Container.Image.Contains("octopusdeploy/worker-tools"))
                                {
                                    # Determine architecture
                                    if ($action.Container.Image.Contains("ubuntu") -and ($action.Container.Image -ne $linuxWorkerToolsImage))
                                    {
                                        # Update with specified image
                                        Write-Host "Updating step $($step.Name) to use $linuxWorkerToolsImage ..."
                                        $action.Container.Image = $linuxWorkerToolsImage
                                        $processUpdated = $true
                                    }


                                    if ($action.Container.Image.Contains("windows") -and ($action.Container.Image -ne $windowsWorkerToolsImage))
                                    {
                                        # Update with specified image
                                        Write-Host "Updating step $($step.Name) to use $windowsWorkerToolsImage ..."
                                        $action.Container.Image = $windowsWorkerToolsImage
                                        $processUpdated = $true
                                    }


                                    if (!$processUpdated)
                                    {
                                        Write-Host "Step $($step.Name) already using specified image, skipping ..."
                                    }
                                }
                                else
                                {
                                    Write-Host "Specified container image does not match 'octopusdeploy/worker-tools', skipping ..."
                                }
                            }
                        }
                    }
                }
                else
                {
                    Write-Host "Runbook $($runbook.Name) doesn't have a published snapshot, skipping ..."
                    Continue
                }


                # Create new runbook snaphot
                $runbookSnapshotTemplate = Get-OctopusItems -OctopusUri "$octopusURL/api/$($space.Id)/runbookprocesses/$($publishedRunbookSnapshot.FrozenRunbookProcessId)/runbookSnapshotTemplate" -ApiKey $octopusAPIKey


                $body = @{
                    ProjectId = $project.Id
                    RunbookId = $runbook.Id
                    Name = $runbookSnapshotTemplate.NextNameIncrement
                    Notes = $null
                    SelectedPackages = @()
                }


                                       




                # Include latest built-in feed packages
                foreach($package in $runbookSnapshotTemplate.Packages)
                {
                                            # Get latest package version
                    $packages = Invoke-RestMethod -Uri "$octopusURL/api/$($space.Id)/feeds/$($package.FeedId)/packages/versions?packageId=$($package.PackageId)&take=1" -Headers $header
                    $latestPackage = $packages.Items | Select-Object -First 1
                    $package = @{
                        ActionName = $package.ActionName
                        Version = $latestPackage.Version
                        PackageReferenceName = $package.PackageReferenceName
                    }
       
                    $body.SelectedPackages += $package


                }


                $body = $body | ConvertTo-Json -Depth 10


                Write-Host "Updating runbook process for $($runbook.Name) ..."
                $runbookProcess.Version = $currentRunbookProces.Version
                $runbookProcess.Id = $runbook.RunbookProcessId
                Invoke-RestMethod -Method Put -Uri "$octopusURL/api/$($space.Id)/runbookprocesses/$($runbookProcess.Id)" -Body ($runbookProcess | ConvertTo-Json -Depth 10) -Headers $header


                Write-Host "Creating new snapshot for $($runbook.Name) ..."
                $newRunbookSnapshot = Invoke-RestMethod -Method Post -Uri "$octopusURL/api/$($space.Id)/runbookSnapshots" -Body $body -Headers $header




                Write-Host "Publishing snapshot $($newRunbookSnapshot.Id) ..."
                $runbook.PublishedRunbookSnapshotId = $newRunbookSnapshot.Id
                Invoke-RestMethod -Method Put -Uri "$octopusURL/api/$($space.Id)/runbooks/$($runbook.Id)" -Body ($runbook | ConvertTo-Json -Depth 10) -Headers $header
            }
        }
    }
}
```


## Conclusion


Maintaining deployment processes is typically a non-issue and is usually done on an as-needed basis.  Execution Containers is one of the few areas where you might need to update multiple items which would prove tedious if you have to do it manually.  Thankfully, the Octopus API is robust and can be used to make mass updates in an automated fashion.


Happy deployments!



