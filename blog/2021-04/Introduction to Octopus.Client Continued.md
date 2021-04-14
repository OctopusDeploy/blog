# Introduction to Octopus.Client Continued


Key: 

*Things in italic are to be revised as they may be repetitive against the previous doc.*



@() is an array and read only in PowerShell!





The below blog post will directly continue on from my previous guide [Introduction to Octopus.Client]() and build on some of the concepts presented whilst identifying some further pitfalls and issues you may encounter along the way.

This guide, like the previous, is intended as a beginner guide to working with the Octopus.Client and uses fairly basic PowerShell scripting. This is to avoid over-complicating things for new users, and for the simple fact that I am rather bad at PowerShell.

The goal of this guide is to expand on the concepts I outlined in the first guide and to attempt to provide some solutions for problems I have invented but may exist for you in some way when using Octopus. At the very least, you will get a good idea of what is possible with the Octopus.Client and how you can automate some daunting tasks on your own instances.



In the previous guide I covered the following:

* Locking Tentacle Upgrade for a single target.
* Modifying the body of a script step.
* Creating a Channel for a single project.

In this guide, I will be covering the following:

* Locking Tentacle upgrade for all targets with active deployments in the specified environment.
* Creating a Channel in each project which deploys a specific package.
* Assigning Tenant tags to any Tenants connected to specific projects.



Now is a good time to check out the [Getting Started]() section of my previous guide if you have not yet used the Octopus.Client. It covers setting up your environment and making sure everything works.

### Tentacle Upgrade Lock expanded.

Previously, we locked a single Tentacle from upgrading by selecting a machine and editing the `UpgradeLocked` value nested inside of it. In this script, we will automate this task for any machines that meet our criteria. For this example, I am going to lock the Tentacle upgrade for all machines which have an active deployment in my "Prod" environment. To clarify, Octopus uses the dashboard to list "active" deployments. Active in this case means it is the latest successful deployment of the latest release into the specified environment.

Why is this script useful?

* It introduces you to navigating the dashboard via the API
* It allows you to lock the upgrade for all machines with an active release based on environment.
* Even if this example is not very useful to you, it shows some methods you can use to get started with the Octopus.Client.



With that, let's get started.



As the goal of this script is to lock all Tentacles with active production deployments, my first step will be to try and identify which active deployments we have. To do this, I will reference the Octopus Dashboard with the following command. Adding the `.Items` at the end will return each individual item on the dashboard.

```powershell
$client.Repository.Dashboards.GetDashboard().Items
```

This is an example of a dashboard item from my local Octopus server. It helps to get any object you intend to work with to better familiarize yourself with the data it contains. I can see it contains a list of helpful information about this particular deployment. I can see it belongs to  `Projects-2`, was deployed to `Environments-22`, it succeeded (`State : Success`), and so on. Whilst these ID's are not too memorable, we can run some convenient commands to help us reference them by their names.

```json
ProjectId               : Projects-2
EnvironmentId           : Environments-22
ReleaseId               : Releases-21
DeploymentId            : Deployments-24
TaskId                  : ServerTasks-560
TenantId                : 
ChannelId               : Channels-2
ReleaseVersion          : 0.0.1
Created                 : 8/02/2021 10:11:32 AM +10:00
QueueTime               : 8/02/2021 10:11:32 AM +10:00
StartTime               : 8/02/2021 10:11:32 AM +10:00
CompletedTime           : 8/02/2021 10:11:40 AM +10:00
State                   : Success
HasPendingInterruptions : False
HasWarningsOrErrors     : False
ErrorMessage            : 
Duration                : 8 seconds
IsCurrent               : True
IsPrevious              : False
IsCompleted             : True
Id                      : Deployments-24
LastModifiedOn          : 
LastModifiedBy          : 
Links                   : {[Self, /api/Spaces-1/deployments/Deployments-24], [Release, /api/Spaces-1/releases/Releases-21], [Tenant, /api/Spaces-1/tenants/], [Task, /api/tasks/ServerTasks-560]}

```



The first thing I would like to do here is create a variable to define the `Prod` environment in our script. Since majority of the time I won't know the ID of a resource, performing a `FindByName` works wonders.

```powershell
$envName = $client.Repository.Environments.FindByName("Prod").Id
```

Once we have defined a variable to hold our desired `Environments-Id`, we will need to get a list of items from the dashboard where the `Environments-Id` matches our new `$envName` variable. This is done by getting the dashboard items as we did before and piping the results through a [Where-Object](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/where-object?view=powershell-7.1). 


So we will: Get all dashboard items | where the `Environments-Id` directly matches our `$envName` value. 

```powershell
$deployments = $client.Repository.Dashboards.GetDashboard().Items | ? {$_.EnvironmentId -match $envName}
```


The `$deployments` variable should now return a list of dashboard items which have an active release in our `Prod` environment. The next step is to try and find any `Machines-Id`'s associated with this deployment. To do so, we can use the deployment variable which directly references a `Deployments-Id`. In the above example, this is `Deployments-24`, so I'll need to GET that deployment and see which machines it was executed against.

First I create a new [array](https://docs.microsoft.com/en-us/powershell/scripting/learn/deep-dives/everything-about-arrays?view=powershell-7.1) to store our list of `Machines-Id`'s. Next I loop through our `$deployments` variable and pass the `Deployments-Id` for each one into a GET for the deployments repository on the Octopus server. This repository contains some more detailed information about each deployment, including any `Machines-Id`'s it deployed to. 

```powershell
$activeMachines = @()

foreach ($deploy in $deployments) {$activeMachines += $client.Repository.Deployments.Get($deploy.id).DeployedToMachineIds}
```



If everything has gone according to plan so far, you should have an `$activeMachines` variable with a list of `Machine-Is`'s that the dashboard recognizes as having active deployments in the Prod environment. The next step is to iterate over each machine in this list and apply the same Tentacle `UpgradeLocked` process we used in my previous guide.

```powershell
    foreach ($machineToLock in $activeMachines) { 
    $machine = $client.Repository.Machines.Get($machineToLock)
    $machine.Endpoint.TentacleVersionDetails.UpgradeLocked = $False
    $client.Repository.Machines.Modify($machine)
}
```



Here is the complete script all together. Its not very complex but does a good job at performing a bulk configuration change against all of my active production targets. 

```powershell
$envName = $client.Repository.Environments.FindByName("Prod").Id
$deployments = $client.Repository.Dashboards.GetDashboard().Items | ? {$_.EnvironmentId -match $envName}

$activeMachines = @()

foreach ($machines in $deployments) {$activeMachines += $client.Repository.Deployments.Get($machines.id).DeployedToMachineIds}

foreach ($machineToLock in $activeMachines) {
    $machine = $client.Repository.Machines.Get($machineToLock)
    $machine.Endpoint.TentacleVersionDetails.UpgradeLocked = $False
    $client.Repository.Machines.Modify($machine) | out-null
    Write-Host $machineToLock "is locked?" $machine.Endpoint.TentacleVersionDetails.UpgradeLocked
}
```

You may never need to perform this specific task, but hopefully this has helped you better understand how you can identify and interact with a particular set of machines and use the dashboard to your advantage.



### Create Channel for multiple projects

In the previous guide, we created a Channel for a single project and talked about some of the ways that Octopus expects its data to be presented. This involved creating some new objects, and assigning a minimum amount of data to satisfy Octopus. *This minimum amount of required data is essentially the same as manually creating something via the portal and getting an error that you have missed some field/setting. So creating a test object via the UI is often a great way to confirm just what is needed as a minimum.*

This script will check all of your projects to see if any of the steps use a specific `PackageId` in any of their package steps. If they do, create a Channel in that project with whatever settings we want. 

Why is this script useful?

* You can create a Hotfix channel for any projects that uses a specific package.
* You can easily change the creation to be conditional on some other value, I just used package for my example.
* If you have a lot of projects and need to dynamically create any Channels, this script can be a good place to start.



First, I want to define my `PackageID` so I know which projects to add my Channel to. I will also need to get a list of all projects. The following two lines will create two variables, one for our target package, and one for all our projects.

```powershell
$packageId = "json" # Yes, I have a package nammed "json".

$projects = $client.Repository.Projects.GetAll()
```



Next, I will need to iterate through all of my projects and for each one, dig into the deployment process to see if it contains steps which are using our defined package. The logic looks like this:

For each step in the deployment process of each project -> if a package exists and is of the same ID as our `$packageId` variable -> we should do a thing. (Create our Channel.)

```powershell
foreach ($project in $projects){
    $process = $client.Repository.DeploymentProcesses.Get($project.deploymentProcessId)
    
    foreach ($step in $process.Steps){
            $packages = $step.Actions.Packages
            
            if ($null -ne $packages){
                $packageIds = $packages | Where-Object {$_.PackageId -eq $packageId}
                
                if($packageIds.Count -gt 0) {
```



This next part will be essentially the same as I did in the previous guide, with a couple of small changes. 

First, I can easily just use the `$project.Id` from the outer `foreach` loop we're currently working in to programmatically define the `ProjectId` we want to attach our Channel to. 

Next, I can use `$step.Name` when telling the Channel which package step to associate with the Channel using the second `foreach` loops `$step` variable.

```powershell
$newChannel = New-Object Octopus.Client.Model.ChannelResource
$newChannel.Name = "Hotfix channel"
$newChannel.Description = "Channel for emergency hotfix releases. Created via API script."
$newchannel.ProjectId = $project.id
$newChannel.SpaceId = "Spaces-1" 

$newChannelRules = New-Object Octopus.Client.Model.ChannelVersionRuleResource
$newChannelRules.ActionPackages.Add($step.Name)
$newChannelRules.VersionRange = "[1.0.0, 1.1.0]" 
$newChannelRules.Tag = "hotfix"

$newChannel.Rules = $newChannelrules
```



And here is the complete script for creating a Channel based on the presence of our defined `$packageId` value within a project.

```powershell
$packageId = "json"

$projects = $client.Repository.Projects.GetAll()

foreach ($project in $projects){
    $process = $client.Repository.DeploymentProcesses.Get($project.deploymentProcessId)
    
    foreach ($step in $process.Steps){
            $packages = $step.Actions.Packages
            
            if ($null -ne $packages){
                $packageIds = $packages | Where-Object {$_.PackageId -eq $packageId}
                
                if($packageIds.Count -gt 0){
                
                    $newChannel = New-Object Octopus.Client.Model.ChannelResource 
                    $newChannel.Name = "Hotfix channel"
                    $newChannel.Description = "Channel for emergency hotfix releases. Created via API script."
                    $newchannel.ProjectId = $project.id
                    $newChannel.SpaceId = "Spaces-1" 

                    $newChannelRules = New-Object Octopus.Client.Model.ChannelVersionRuleResource
                    $newChannelRules.ActionPackages.Add($step.Name) 
                    $newChannelRules.VersionRange = "[1.0.0, 1.1.0]" 
                    $newChannelRules.Tag = "hotfix"

                    $newChannel.Rules = $newChannelrules

                    $client.Repository.Channels.Create($newChannel)
                }
            }
    }
}
```

Once again, you may never need to use this specific example in your Octopus adventures. However, I think it contains some useful concepts that you could very well benefit from when interacting with the Octopus API.



### Add Tenant tags to specific Tenants

The final script in this guide will loop through all Tenants connected to a specified project and add a list of Tenant tags if they don't exist on the Tenant.

Why is this script useful?

* You can quickly add new Tenant tags to a set of existing Tenants, assuming there is some way to distinguish them. (I'm using connected projects)
* Introduces working with [PowerShell hashtables](https://docs.microsoft.com/en-us/powershell/scripting/learn/deep-dives/everything-about-hashtable?view=powershell-7.1) in the Octopus.Client.
* It is a great platform for any bulk Tenant actions you may want to use the API for.



To begin this script, I need to set a few arrays and a variable containing a list of my Tenants. I will need a list of Tenant tags that I would like to add to my desired Tenants, and a way to select which Tenants they will be added to. I am using connected projects to identify my desired Tenants. 

My script starts as below:

```powershell
$tagsToAdd = @(
                'tagset-A/Tag One',
                'tagset-A/Tag Two'
              )

$projectsToAdd = @(
                    'abc',
                    'test',
                  )
$projects = @()

$tenants = $client.Repository.Tenants.FindAll()
```

I have an array containing two tags from the same tag set. This script should work with any amount or variation of tag sets and tags. I also have an array containing two projects that I want to use as my reference. If a Tenant is connected to either of these projects, add the tags. I will need an empty array to store the `Project-Id`'s associated with our project names. This step once again is to save having to manually find each `Projects-Id`. Finally, I am setting a Tenants variable to get all Tenants for me to iterate through.


The next step is to populate our empty `$projects` array with the `Projects-Id` value of each project name listed in our `$projectsToAdd` array.

```powershell
foreach ($project in $projectsToAdd){
    $projects += $client.Repository.projects.FindByName($project).id
}
```

With that, we are ready for the body of this script. So the logic should go like this:

For each tenant in our list of tenants -> Iterate over the `projectEnvironments` using the [.GetEnumerator()](https://docs.microsoft.com/en-us/powershell/scripting/learn/deep-dives/everything-about-hashtable?view=powershell-7.1#iterating-hashtables) function (This is required as `ProjectEnvironments` is a hashtable) -> For each object in this Tenant's `ProjectEnvironments` -> If the key value is present in our projects array, execute our code.

```powershell
foreach ($tenant in $tenants){
    $tenant.ProjectEnvironments.GetEnumerator() | ForEach-Object {
        if ($_.key -in $projects){
```



I'm using a further `foreach` here to iterate over the list of tags we have, and to add them to the target if they are not already there. 

The logic: `foreach` tag in our array -> If it is not present on the Tenant -> Use the add function to add it -> If it already exists -> Write some message to say so -> else if it both does exist and doesn't exist, something very unexpected has gone wrong and you should call your local quantum physicist. :)

The final part of our script will be to modify the Tenant with our updated Tenant tags, remembering that we need to save each entire Tenant object individually. I do this by placing the modify at the end of each outer loop where we are iterating over the individual Tenants.

```powershell
            foreach ($tag in $tagsToAdd){
                if ($tenant.TenantTags -notcontains $tag){
                    $tenant.TenantTags.add($tag)
                }
                elseif ($tenant.TenantTags -contains $tag) {
                    Write-Host $tenant.TenantTags "already exists on" $tenant.Name
                }
                else {
                    Write-Host "Something unexpected must have gone wrong :("
                }
            }
        }
    }
    $client.Repository.Tenants.Modify($tenant)
}
```



And here is the entire script. I have removed the `else` seen in the above code block as it is redundant, the Tenant tags either exist on the Tenant, or they don't. 

```powershell
$tagsToAdd = @(
                'tagset-A/Tag One',
                'tagset-A/Tag Two'
              )

$projectsToAdd = @(
                    'abc',
                    'test',
                  )
$projects = @()

$tenants = $client.Repository.Tenants.FindAll()


foreach ($project in $projectsToAdd){
    $projects += $client.Repository.projects.FindByName($project).id
}

foreach ($tenant in $tenants){
    $tenant.ProjectEnvironments.GetEnumerator() | ForEach-Object {
        if ($_.key -in $projects){
            foreach ($tag in $tagsToAdd){
                if ($tenant.TenantTags -notcontains $tag){
                    $tenant.TenantTags.add($tag)
                }
                elseif ($tenant.TenantTags -contains $tag) {
                    Write-Host $tenant.TenantTags "already exists on" $tenant.Name
                }        
            }
        }
    }
    $client.Repository.Tenants.Modify($tenant)
}
```



## Conclusion

I think it's safe to say that there are many different ways you can use the Octopus.Client and many more ways you can interact with Octopus via the API. It is, after all, an API first program, and I'm only showing examples in PowerShell. 

In this second guide, I have attempted to expand on some of the concepts covered in my initial guide, whilst providing some potentially practical uses for the Octopus.Client focusing on performing bulk server tasks. Each script presented here will probably not have an exact use case for you, but they can be useful for practice the concepts and methods commonly employed when interacting with Octopus via the API.

If you have any suggestions on what you would like to see in future posts, or have some feedback on my amateur PowerShell scripting, please feel free to leave a comment.

