## Introduction to Octopus.Client

Octopus has always been designed and advertised as an API first program, though our experience for new users dipping their toes in the API can be daunting and confusing. There are multiple different approaches you can take, be it the Octopus.Client.dll, our [SwaggerUI](https://octopus.com/docs/octopus-rest-api#rest-api-documentation-via-swagger), [REST](https://octopus.com/docs/octopus-rest-api), or just poking around the /api/ endpoint on your Octopus web portal.

In the following guide, I'm going to attempt to outline some basic principles of manipulating your Octopus server via the API. I will start by using the Octopus.Client with PowerShell, though in later guides I may also cover CURL and C#.

This guide contains three practice examples to help you get started with the Octopus.Client in PowerShell. By the end of this tutorial, you will know how to use the Octopus.Client to lock and unlock Tentacles, modify the script body in a script step, and create a new Channel for a project. Each stage of this tutorial will build of the previous to progressively perform more complex API actions.

Why should you bother with the Octopus API when everything can be accomplished via the web portal?

* Perform dynamic bulk tasks.
* Automate any manual task in Octopus.
* Provide reporting on esoteric data throughout your instance.

The [Octopus.Client](https://octopus.com/docs/octopus-rest-api/octopus.client) is a .NET library that makes it easy to write C# programs or PowerShell scripts that manipulate your Octopus server. Furthermore, it's completely [open source](https://github.com/OctopusDeploy/OctopusClients).

We are also currently in the process of building a GO client with future plan to tackle other platforms.

Finally, if you're not looking to play around with scripting, check out our [Octopus CLI](https://octopus.com/docs/octopus-rest-api/octopus-cli). It contains a load of handy functions to help you automate Octopus without any programming or scripting knowledge and can be easily called during your deployments.

### Getting started.
There are a few prerequisites to following this guide. To start, you will need to ensure you have the following resources available.

* PowerShell.

* Octopus.Client.dll (Can be found [online here](https://www.nuget.org/packages/Octopus.Client/), or on your Octopus server installation directory, default `C:\Program Files\Octopus\Octopus Server\Octopus.client.dll`)

* Octopus server instance.

* [API Key](https://octopus.com/docs/octopus-rest-api/how-to-create-an-api-key) for account with appropriate access.  (Project contributor & Environment manager are a good fit here.)

* An [Environment](https://octopus.com/docs/infrastructure/environments) with at least one Tentacle ([Windows](https://octopus.com/docs/infrastructure/deployment-targets/windows-targets) or [Linux](https://octopus.com/docs/infrastructure/deployment-targets/linux/tentacle)) configured and working.

* A new [Project](https://octopus.com/docs/projects) for testing with a [Script Step](https://octopus.com/docs/deployments/custom-scripts/run-a-script-step) and [Package Step](https://octopus.com/docs/deployments/packages#adding-a-package-step).

#### Setting up your environment

When you have prepared your environment as per the above instructions, you should be ready to start. The first thing we need to do is locate our `Octopus.Client.dll` and load it into PowerShell. In my setup, I have a `C:\MyScripts` folder containing the `Octopus.Client.dll`.


To begin, you need to run the `Add-Type` command and direct PowerShell to the `Octopus.Client.dll`. Though commented out, I have provided the command to check the version of your `Octopus.Client.dll`.

```powershell
Add-Type -Path 'C:\MyScripts\Octopus.Client.dll'
#[System.Diagnostics.FileVersionInfo]::GetVersionInfo('C:\MyScripts\Octopus.Client.dll').FileVersion
```

Next, you will need to set a variable to authenticate with the Octopus server using your API key.

```powershell
$endpoint = New-Object Octopus.Client.OctopusServerEndpoint("http://YourOctopusServer/", "API-SBACIO3TKYAGSOTVNMYWYTRTYA")
$repository = New-Object Octopus.Client.OctopusRepository($endpoint)
```

Note: Octopus will always use the default [Space](https://octopus.com/docs/administration/spaces) unless you explicitly define a space. By adding the following lines, you can target your scripts at a different Space. If you have not created a Space, or are working on your default Space, you can skip this step.

```powershell
# Spaces declaration

$repository = $client.ForSystem()
$space = $repository.Spaces.FindByName("Space Name")

$repositoryForSpace = $client.ForSpace($space)
```

If you are working with Spaces, the `$respositoryForSpace` variable will replace the use of `$repository` throughout this guide.


#### Now for a quick test

First, let's make sure everything is working as expected.

I am going to try and find a deployment target on my instance named `Machine-A`. For context, I know this to be a *Windows Listening Tentacle*.

```powershell
$repository.machines.FindByName("Machine-A")
```

PowerShell should return the details for the target named `Machine-A`. This is the same information you would see if you were to search https://YourOctopusServer/api/machines/Machines-1 in your browser.

```text
EnvironmentIds                  : {Environments-1}
Roles                           : {web}
TenantedDeploymentParticipation : Untenanted
TenantIds                       : {}
TenantTags                      : {}
Name                            : Machine-A
Thumbprint                      : C89B53CCF1771F390421D439F288ED026498B433
Uri                             : https://localhost:10933/
IsDisabled                      : False
MachinePolicyId                 : MachinePolicies-1
Status                          : Online
HealthStatus                    : Healthy
HasLatestCalamari               : True
StatusSummary                   : Octopus was able to successfully establish a connection with this machine on Tuesday, 29 September 2020 
                                  10:11:31 AM +10:00
IsInProcess                     : False
Endpoint                        : Octopus.Client.Model.Endpoints.ListeningTentacleEndpointResource
Id                              : Machines-1
LastModifiedOn                  : 
LastModifiedBy                  : 
Links                           : {[Self, /api/Spaces-1/machines/Machines-1], [Connection, /api/Spaces-1/machines/Machines-1/connection], 
                                  [TasksTemplate, /api/Spaces-1/machines/Machines-1/tasks{?skip,take}]}
```



### Baby steps
Now that we have tested and confirmed that our client and server working, and we're authenticated, let's try some basic API manipulation. If you are stuck here and find yourself unable to progress, feel free to get in touch with [Octopus Support](https://octopus.com/support).

#### Tentacle Upgrade Lock

My first task is going to be locking the upgrade for our Tentacle named `Machine-A`. Locking the upgrade of a Tentacle will stop Octopus from upgrading to later Tentacle versions until it is unlocked again.

Why would I do this?

* You can use this script to lock a single Tentacle based on name or as a template to build a script that will allow you to lock all Tentacles within your defined parameters. (More on this in my next blog post.)

Looking at the above data for the `Machine-A` object, there is no value presented for us to lock the Tentacle. This value is actually nested inside of the `Endpoint` object. You can see above that this is a nested object `Octopus.Client.Model.Endpoints.ListeningTentacleEndpointResource`, so we will need to see what is inside of the Endpoint object to continue.

To start, I will need to set a variable (`$machine`) to hold our `Machine-A` object while we dig through it and make our changes. I can then inspect the content of `$machine.Endpoint`, see below for the results.

```powershell
$machine = $repository.machines.FindByName("Machine-A")
$machine.Endpoint

Uri                           : https://Localhost:10933
ProxyId                       : 
Thumbprint                    : C89B53CCF1771F390421D439F288ED026498B433
TentacleVersionDetails        : Octopus.Client.Model.Endpoints.TentacleDetailsResource
CertificateSignatureAlgorithm : sha256RSA
Id                            : 
LastModifiedOn                : 
LastModifiedBy                : 
Links                         : {}
```


It looks like we're getting somewhere, we can see `TentacleVersionDetails`. Again, I can simply append this to our `Endpoint` command and we have the following.

```powershell
$machine.Endpoint.TentacleVersionDetails

UpgradeLocked Version UpgradeSuggested UpgradeRequired
------------- ------- ---------------- ---------------
         False 5.0.15             False           False
```


Excellent! I can see that `Machine-A` is running Tentacle version `5.0.15` and the `UpgradeLocked` value is `False`. Since I want to stop this Tentacle from upgrading past `5.0.15`, I will set `UpgradeLocked` to `True`. Once more, I append the resource I want to change to our command. As `UpgradeLocked` is what we need to change, I set it to a value of `$true`.

```powershell
$machine.Endpoint.TentacleVersionDetails.UpgradeLocked = $true
```


Finally, I will confirm the value is set by running the previous command:

```powershell
$machine.Endpoint.TentacleVersionDetails

UpgradeLocked Version UpgradeSuggested UpgradeRequired
------------- ------- ---------------- ---------------
         True 5.0.15             False           False
```


It looks like the new value is set! Time to apply our changes. To do this, we need to save the entire `$machine` object we have been working on to our Octopus server. It is important that you save the entire object you are working on. The `Octopus.Client` won't allow you to only save a part of an object. 

Most repositories inside of the `Octopus.Client` will have a Modify feature.

```powershell
$repository.Machines.Modify($machine)
```


Below is the above script I have stepped through with the modify line commented out. You can run this and confirm that you are able to successfully lock the upgrade for the target, then uncomment the final line to save the changes.

```powershell
Add-Type -Path 'C:\MyScripts\Octopus.Client.dll'
$endpoint = New-Object Octopus.Client.OctopusServerEndpoint("http://YourOctopusServer/", "API-SBACIO3TKYAGSOTVNMYWYTRTYA")
$repository = New-Object Octopus.Client.OctopusRepository($endpoint)

$machine = $repository.Machines.FindByName("Machine-A")
$machine.Endpoint.TentacleVersionDetails.UpgradeLocked = $true
$machine.Endpoint.TentacleVersionDetails
#$repository.Machines.Modify($machine)
```



#### Modify Script Body

Below I will provide steps to edit the script value for a script step inside a project.

Why would I do this?

* You probably won't, but it's a great learning exercise and covers some interesting points.

To get started, let's define a new value we will be applying to the body of our script step.

```powershell
$scriptBody = "Write-Host 'Hello, World!'"
```


Next, we need to identify which project's deployment process we will be editing. Each step inside of your project is a part of a deployment process and each deployment process is identifiable in the API with an ID such as `deploymentprocess-Projects-2`. There are a few ways to find the ID for the deployment process you would like to edit.

Below I will demonstrate a method you can use with the `Octopus.Client`. (My project is named `Hello, World!`)

```powershell
$deploymentProcessId = $repository.Projects.FindByName("Hello, World!").DeploymentProcessId
```


Now that we have a variable holding our deployment process Id, we can search the repository for that specific deployment Process. Note, we will need to hold the entire deployment process in a variable so we can save our changes as a single object as we did in the previous example.

Within our deployment process, you should see something like the following:

```powershell
$deployProcess = $repository.DeploymentProcesses.Get($deploymentProcessId)

ProjectId      : Projects-2
Steps          : {Hello World, Run a Script, Deploy a Package}
Version        : 13
LastSnapshotId : 
SpaceId        : Spaces-1
Id             : deploymentprocess-Projects-2
LastModifiedOn : 
LastModifiedBy : 
Links          : {[Self, /api/Spaces-1/deploymentprocesses/deploymentprocess-Projects-2], [Project, /api/Spaces-1/projects/Projects-2], 
                 [Template, /api/Spaces-1/deploymentprocesses/deploymentprocess-Projects-2/template{?channel,releaseId}]}
```

What I want to do is modify the second step in the above `Steps` object, `Run a Script`. But first, let's see what this `Steps` object returns.

It's a list containing each of our steps. I can't see a reference to the Script body on my `Run a Script` step which means it's probably nested inside of `Properties` or `Actions`.

```powershell
$deployProcess.Steps

Id                           : 50362009-0ea2-466a-8c91-a17006222afa
Name                         : Hello World
RequiresPackagesToBeAcquired : False
PackageRequirement           : LetOctopusDecide
Properties                   : {[Octopus.Action.TargetRoles, Octopus.Client.Model.PropertyValueResource]}
Condition                    : Success
StartTrigger                 : StartAfterPrevious
Actions                      : {Hello World}

Id                           : 4acd97e9-d2e5-48bc-b9b6-6975abd471a9
Name                         : Run a Script
RequiresPackagesToBeAcquired : False
PackageRequirement           : LetOctopusDecide
Properties                   : {[Octopus.Action.TargetRoles, Octopus.Client.Model.PropertyValueResource]}
Condition                    : Success
StartTrigger                 : StartAfterPrevious
Actions                      : {Run a Script}

Id                           : a82e4ce6-e5e5-403f-b001-34ad52be4027
Name                         : Deploy a Package
RequiresPackagesToBeAcquired : False
PackageRequirement           : LetOctopusDecide
Properties                   : {[Octopus.Action.TargetRoles, Octopus.Client.Model.PropertyValueResource]}
Condition                    : Success
StartTrigger                 : StartAfterPrevious
Actions                      : {Deploy a Package}
```


Since each value here is a in a list, I need to identify which object to dig further into. I want to look at the `Actions` object of the `Run a Script` step. So to do this, I point to the index value in the list, which is `[1]`. Since this is a zero indexed list, `[0]` will be our first step `Hello World`.

```powershell
$deployProcess.Steps[1].actions


Name                          : Run a Script
ActionType                    : Octopus.Script
IsDisabled                    : True
WorkerPoolId                  : 
CanBeUsedForProjectVersioning : False
IsRequired                    : False
Environments                  : {}
ExcludedEnvironments          : {}
Channels                      : {}
TenantTags                    : {}
Properties                    : {[Octopus.Action.RunOnServer, Octopus.Client.Model.PropertyValueResource], [Octopus.Action.Script.ScriptSource, 
                                Octopus.Client.Model.PropertyValueResource], [Octopus.Action.Script.Syntax, 
                                Octopus.Client.Model.PropertyValueResource], [Octopus.Action.Script.ScriptBody, ]}
Packages                      : {}
Id                            : c956ae86-e25c-4ea1-bbf1-4ba922d7d3ea
LastModifiedOn                : 
LastModifiedBy                : 
Links                         : {}
```


Great, now I can see that under `Properties` there is an object called `Octopus.Action.Script.ScriptBody`. It sounds like I have found what I'm looking for. Next, I want to set the value of this script body to the variable we created above `$scriptBody`. 

The below command will set the `scriptBody` value for my `Run a Script` step.

```powershell
$deployProcess.steps[1].Actions.Properties.'Octopus.Action.Script.ScriptBody'= $scriptBody 
```


Finally, I will need to modify the entire deployment process object with the change I made. If everything goes as planned, our `Hello, World!` project will have an updated `Run a Script` step with a modified script body.

```powershell
$repository.DeploymentProcesses.Modify($deployProcess)
```



All together, this modification only takes 4 lines and can be expanded on to modify a script body over any and all selected projects you need updated. However, as I noted earlier, this is not something you will likely need to do. If you would like to implement a predefined step over a number of projects, you should check out our [Step Templates](https://octopus.com/docs/projects/custom-step-templates) feature.

```powershell
$scriptBody = "Write-Host 'Hello, World!'"

$deployProcess = $client.Repository.DeploymentProcesses.Get("deploymentprocess-Projects-2")

$deployProcess.steps[1].Actions.Properties.'Octopus.Action.Script.ScriptBody'= $scriptBody

$client.repository.DeploymentProcesses.Modify($deployProcess)
```



#### Create Channel for Project

Finally, I'm going to discuss adding a Channel to your project. This Channel will contain version rules and point to a package step in your project.

Why would I do this?

* You may want to do this if you need to programmatically create new channels for many projects consistently and quickly.

Let's start by getting the ID for the project we want to add our Channel to.

```powershell
$projectId = $repository.Projects.FindByName("Hello, World!").Id
```


Now we need to get started with creating our Channel object. Since there is no existing resource, we need to create one.

 ```powershell
$newChannel = New-Object Octopus.Client.Model.ChannelResource

Name           : 
Description    : 
ProjectId      : 
LifecycleId    : 
IsDefault      : False
Rules          : {}
TenantTags     : {}
SpaceId        : 
Id             : 
LastModifiedOn : 
LastModifiedBy : 
Links          : {}
 ```


Our `$newChannel` object, which I printed out above, will show us with the different fields we can fill. Keep in mind that some objects have further nested objects. Often times when working with the API and creating new objects, It helps to retrieve an existing example created through the UI to get a better idea about the data Octopus expects.

However, I know from experience that I will need a `Name`, `Description`, `ProjectID`, and `SpaceId` as a minimum. Octopus will automatically fill certain fields such as `Id` and `Links`, so we never define an `Id` when creating a resource to help Octopus avoid conflicts.

Let's add some values to our new Channel object. These are expected by Octopus and cannot be skipped. Make note of the Space you are working on so you can configure the value correctly. (Spaces-1 is the "default" Space.)

```powershell
$newChannel.Name = "Panama Channel"
$newChannel.Description = "Good for trade"
$newchannel.ProjectId = $projectId 
$newChannel.SpaceId = "Spaces-1" 
```


So our new Channel is coming together with some basic information, next we will want to configure the important stuff. Version rules, associated package steps, pre-release tags (If you are using any).

If you were to retrieve an existing Channel object, you will notice that the `ChannelVersionRule` is a nested object. So we will need to create a new object for it here.

```powershell
$newChannelRules = New-Object Octopus.Client.Model.ChannelVersionRuleResource
```


Within our Version Rule resource, there is an `ActionPackages` object. This object expects values in a list format, so you can just add the desired package steps by name. I have a single package step named `Deploy a Package` that I want to add to the Channel version rules. Alongside a version range that will pick any package from 1.0.0 to 1.1.0. See our [Documentation on version rules](https://octopus.com/docs/releases/channels#Channels-versionrules). Finally, I define a pre-release tag that I'm calling `example`.

```powershell
$newChannelRules.ActionPackages.Add("Deploy a Package")
$newChannelRules.VersionRange = "[1.0.0, 1.1.0]"
$newChannelRules.Tag = "example"
```


Now to save our new version rule object to the new Channel object.

```powershell
$newChannel.Rules = $newChannelrules
```

As we have in our previous scripts, we need to create the Channel by passing our entire `$newChannel` object into it.

```powershell
$repository.Channels.Create($newChannel)
```


And with that, you should have a new Channel in your project with all the above details.

The full script for this is below:

```powershell
$projectId = $client.Repository.Projects.FindByName("Hello, World!").Id

$newChannel = New-Object Octopus.Client.Model.ChannelResource
$newChannel.Name = "Panama channel"
$newChannel.Description = "Also good for trade"
$newchannel.ProjectId = $projectId
$newChannel.SpaceId = "Spaces-1"

$newChannelRules = New-Object Octopus.Client.Model.ChannelVersionRuleResource
$newChannelRules.ActionPackages.Add("Deploy a Package")
$newChannelRules.VersionRange = "[1.0.0, 1.1.0]"
$newChannelRules.Tag = "example"

$newChannel.Rules = $newChannelrules

$client.Repository.Channels.Create($newChannel)
```



#### Troubleshooting common issues

Below is a list of common *gotchas* that I notice while working with the Octopus.Client. Hopefully this section will take some guess work out of issues you may encounter and allow you to avoid some common mistakes.

* When creating new objects, never provide an ID, Octopus will automatically fill the ID or create one where needed.
* Never specify a version of a resource. If you are modifying a variable of version 5, you do not need to tell Octopus that your modification is now version 6, Octopus will do this automatically. If you do specify this value, you run the risk of corrupting the resource by altering the version incorrectly or altering the version while another user is already editing the data. It's best to let Octopus handle the generation and incrementation of Id's and Versions.
* When making a change via the API, it's important that you change the entire object. As an example, I'm not able to simply edit the script body and push the modified script body object to Octopus. I need to save the entire deployment process to Octopus. The same goes for modifying the Upgrade Locked value on the Tentacle in my first example, I need to save the entire machine object after my changes, not just the nested object containing the changed value.
* Working with the API opens you up to potentially corrupting data if you are not conscious about the edits you make. 
* If in doubt about how a new resource should look, get an existing one and try to copy the structure.



### Conclusion

That concludes my introductory Octopus.Client API tutorial. This tutorial is intended for new users to the Octopus.Client / Octopus API. In my next post I will expand on the above concepts and provide examples for working on a larger scale within Octopus.

For further script examples, visit our [open source Octopus.Client script repository on GitHub](https://github.com/OctopusDeploy/OctopusDeploy-Api/tree/master/Octopus.Client).



