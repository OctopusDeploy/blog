## Introduction to Octopus.Client

Octopus has always been designed and advertised as an API first program, though our experience for new users dipping their toes in the API can be daunting and confusing. There are multiple different approaches you can take, be it the Octopus.Client.dll, our SwaggerUI, REST, or just poking around the /api/ endpoint on your Octopus web portal.

In the following guide, I'm going to attempt to outline some basic principles of manipulating your Octopus server via the API. I will start by using the Octopus.Client and PowerShell, though in later guides I will also cover CURL.

This guide contains three practice examples to help you get started with the Octopus.Client in PowerShell. By the end of this tutorial, you will be able to use the Octopus.Client to lock and unlock Tentacles, modify the script body in a script step, and create a new Channel for a project. Each stage of this tutorial will build of the previous to progressively perform more complex API actions.

Why should you bother with the Octopus API when everything can be accomplished via the web portal?

* Perform bulk tasks.
* Automate any manual task in Octopus.
* Perform many common functions without scripting.

The [Octopus.Client](https://octopus.com/docs/octopus-rest-api/octopus.client) is a .NET library that makes it easy to write C# programs or PowerShell scripts that manipulate the. Furthermore, it's completely [open source](https://github.com/OctopusDeploy/OctopusClients).

We are also currently in the process of building a GO client with future plan to tackle other platforms.

# Feedback

I really like your intro. I’d add a couple things to it though.

1. Mention the Octopus CLI as a handy way to do a lot of common functions without scripting.
2. Mention that we’re building a GO client as well as plan to tackle other platforms too.
3. more on the scope of the document from the start. Scaling up through each part to build on previous info.
4. I’d probably move the trouble shooting section before the conclusion. Minor point.
   

### Getting started.
We cover this very well in our documentation but I'll summarize and recap here.
Where can they download/find the Client?
Where can they see the version and revisions?

There are a few prerequisites to following this guide, you will need to ensure you have the following resources available.

* PowerShell.

* Octopus.Client.dll (Can be found on your Octopus server installation directory, default `C:\Program Files\Octopus\Octopus Server\Octopus.client.dll`)

* Octopus server instance

* [API Key](https://octopus.com/docs/octopus-rest-api/how-to-create-an-api-key) for account with appropriate access (What is this access? Do I list permissions?)

* An environment with at least one Tentacle configured and working.

* A project with a script step and package step.

#### Setting up your environment

When you have prepared your environment as per the above instructions, you should be ready to start. The first thing we need to do is locate our Octopus.Client.dll and load it into PowerShell. In my setup, I have a MyScripts folder containing the Octopus.Client.dll.



To begin, you need to run the Add-Type command and direct PowerShell to the Octopus.Client.dll.

```powershell
# Locate and add the Octopus.Client.dll to your PowerShell session.
Add-Type -Path 'C:\MyScripts\Octopus.Client.dll'
# Make sure you have a recent version! Here is how to check the loaded Octopus.Client.dll version...
#[System.Diagnostics.FileVersionInfo]::GetVersionInfo('C:\MyScripts\Octopus.Client.dll').FileVersion
```

Next, set a variable to authenticate with the Octopus server using your API key.

```powershell
# Create endpoint and client.
# This uses the above Octopus.Client.dll to create an authenticated endpoint at your Octopus server and a client variable containing the API resources you will call on.
$endpoint = New-Object Octopus.Client.OctopusServerEndpoint("http://YourOctopusServer/", "API-SBACIO3TKYAGSOTVNMYWYTRTYA")
$repository = New-Object Octopus.Client.OctopusRepository($endpoint)
```

Note: Octopus will always use the default space unless you explicitly define a space. By adding the following lines, you can target your scripts at a different space. If you have not created a space, you can skip this step.

```powershell
# Spaces declaration
# Octopus will always act against the default space. A general rule is that Octopus will use default instances unless otherwise stated. This applies for things like Tentacle/Server Instance, etc.
#$repository = $client.ForSystem()
#$space = $repository.Spaces.FindByName("Space Name")
# Get space specific repository and get all projects in space - From here you can just call $repositoryForSpace like $client, it will act the same but return only resources for the defined space.
#$repositoryForSpace = $client.ForSpace($space)
```



#### Now for a quick test

Make sure it all runs fine, what should you see to confirm it's working?

You can confirm that you have everything working by running the below command.

PowreShell should return the details for the target named Machine-A. This is the same information you would see if you were to search https://YourOctopusServer/api/machines/Machines-1

```powershell
$repository.machines.FindByName("Machine-A")

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
Now that we have our client and server working, and we're authenticated, let's try some basic API manipulation. 

#### Tentacle Upgrade Lock

My first task is to lock the upgrade for our Tentacle Machine-A. Locking the upgrade of a Tentacle will stop Octopus from upgrading to later Tentacle versions until it is unlocked again.

Why would I do this?

* You can use this script to lock a single Tentacle based on name or as a template to build a script that will allow you to lock all Tentacles within your defined parameters. (More on this in a future post.)

Looking at the above data for the Machine-A object, there is nowhere clearly identified where you can lock the Tentacle. This value is actually nested inside of the `Endpoint` object. You can see above this is a nested object `Octopus.Client.Model.Endpoints.ListeningTentacleEndpointResource`, we will need to see what is inside of the Endpoint object to continue.

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

It looks like we're getting somewhere, we can see TentacleVersionDetails. Let's see what is inside the `Octopus.Client.Model.Endpoints.TentacleDetailsResource`. 



```powershell
$machine.Endpoint.TentacleVersionDetails

UpgradeLocked Version UpgradeSuggested UpgradeRequired
------------- ------- ---------------- ---------------
         False 5.0.15             False           False
```

So here I can see that Machine-A is running Tentacle version 5.0.15 and the `UpgradeLocked` value is `False`. Since I want to stop this Tentacle from upgrading past 5.0.15, I will set `UpgradeLocked` to `True`.

```
$machine.Endpoint.TentacleVersionDetails.UpgradeLocked = 1 ## how to do this with True/False, bug or powershell skills lacking?
```

Finally, I will confirm the value is set by running the previous command:

```powershell
$machine.Endpoint.TentacleVersionDetails

UpgradeLocked Version UpgradeSuggested UpgradeRequired
------------- ------- ---------------- ---------------
         True 5.0.15             False           False
```

And to apply our changes, we need to save the `$machine` object we have been working on to our Octopus server. Most repositories inside of the Octopus.Client have a Modify feature.

```powershell
$repository.Machines.Modify($machine)
```

Below is the above script I have stepped through with the modify line commented out. You can run this and confirm that you are able to successfully lock the upgrade for the target, then uncomment the final line and make the changes.

```powershell
Add-Type -Path 'C:\MyScripts\Octopus.Client.dll'
$endpoint = New-Object Octopus.Client.OctopusServerEndpoint("http://YourOctopusServer/", "API-SBACIO3TKYAGSOTVNMYWYTRTYA")
$repository = New-Object Octopus.Client.OctopusRepository($endpoint)

$machine = $repository.Machines.FindByName("Machine-A")
$machine.Endpoint.TentacleVersionDetails.UpgradeLocked = 1
$machine.Endpoint.TentacleVersionDetails
#$repository.Machines.Modify($machine)
```



#### Modify Script Body

Below I will provide steps to edit the script value in a script step inside a project.

Why would I do this?

* You many have a script that requires updating in many projects. This will give you the basic steps required to later automate that process over any set of projects you define. (More on large scale manipulation in a future post.)

To get started, let's define a new value we will be applying to our script step.

```powershell
$scriptBody = "Write-Host 'Hello, World!'"
```

Next, we need to identify which deployment process we will be editing. Each step inside of your project is a part of a deployment process and each deployment process is identifiable in the API with an ID such as `deploymentprocess-Projects-2`. There are a few ways to find the ID for the deployment process you would like to edit, below I will demonstrate a method you can use with the API.

```powershell
$deploymentProcess = $repository.Projects.FindByName("Hello, World!").DeploymentProcessId
deploymentprocess-Projects-2 # Deployment Process ID for my Hello, World! Project.
```

Now that we have a variable holding our deployment process ID, we can create a variable containing our `Hello, World!` project's deployment Process.

Within our deployment process, you should see something like the following:

```powershell
$deployProcess = $repository.DeploymentProcesses.Get($deploymentProcess)

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

What I want to do is dig into my steps and modify the second step in the above `Steps` object, `Run a Script`. But first, let's see what this `Steps` object returns.

It's a list containing each of our steps. I can't see a reference to the Script body on my `Run a Script` step which means it's probably nested inside of Properties or Actions.

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

Since each value here is a in a list, I need to identify which object to dig further into. I want to look at the `Actions` object of the `Run a Script` step. So to do this, I point to the index value in the list. `[1]` since this is a zero indexed list, `[0]` will be our first step `Hello World`.

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

So I can see that under the Properties object there is an object called `Octopus.Action.Script.ScriptBody`. I want to set the value of this script body to the variable we created above with our script content. The below command will set the `scriptBody` value for my `Run a Script` step.

```powershell
$deployProcess.steps[1].Actions.Properties.'Octopus.Action.Script.ScriptBody'= $scriptBody 
```

Finally, I will need to modify the entire deployment process object with the small change we made. If everything goes as planned, our `Hello, World!` project will have an updated `Run a Script` step with a modified script body.

```powershell
$repository.DeploymentProcesses.Modify($deployProcess)
```



All together, this modification only takes 4 lines and can be expanded on to modify a script body over any and all selected projects you need updated. More on bulk modifications in a future post.

```powershell
# Create a new Scriptbody for our step.
$scriptBody = "Write-Host 'Hello, World!'"

# Call the deployment process and assign it to a variable. You need the ID for this! I'll show to to find it. Alternatives include find-by-name to get the Deployment Process ID.
$deployProcess = $client.Repository.DeploymentProcesses.Get("deploymentprocess-Projects-2")
# Here I'm calling the second step ([1] in a zero index array). I'm then burrowing into the properties to get to the scriptbody object and assigning it the new value.
$deployProcess.steps[1].Actions.Properties.'Octopus.Action.Script.ScriptBody'= $scriptBody  # two scriptbody in the repo?
# Now we just push the new deployment process with a modify and it's done!
$client.repository.DeploymentProcesses.Modify($deployProcess)

```



#### Create Channel for Project

Finally, I'm going to discuss adding a Channel to your project. This channel will contain version rules and point to a package step in your project.

You may want to use this if you need to programmatically create new channels for many projects consistently and quickly.



Jumping into this, let's start by getting the ID for the project we want to add our Channel to.

```powershell
$projectId = $repository.Projects.FindByName("Hello, World!").Id
```

Now we need to get started with creating our Channel Object. Since there is no existing resource, we need to create one.

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

Our $newChannel object will now provide us with the different fields we can fill. Some objects have nested objects. Often times when working with the API and creating new objects, It helps to retrieve an existing example to get a better idea about the data Octopus expects.

I know from experience that I will need a Name, Description, ProjectID, and SpaceId as a minimum. Octopus will automatically fill certain fields such as Id and Links.



Let's add some values to our new Channel object.

```powershell
$newChannel.Name = "Panama channel"
$newChannel.Description = "Good for trade"
$newchannel.ProjectId = $projectId  # It's important to associate this Channel with a project.
$newChannel.SpaceId = "Spaces-1"  # Default space is always Spaces-1, this needs to be defined in this case.
```

So our new Channel is coming together with some basic information, next we will want to configure the important stuff. Version rules, associated package steps, tags (if any)



If you were to retrieve an existing Channel object, you will notice that the ChannelVersionRule is a nested object. So we will need to create one here.

```powershell
$newChannelRules = New-Object Octopus.Client.Model.ChannelVersionRuleResource
```

Within our Version Rule resource, there is an ActionPackage object. This object expects values in a list format, so you can just add the desired package steps by name. I have a single package step named `Deploy a Package` that I want to add to the Channel version rules. Alongside a version range that will pick any package from 1.0.0 to 1.1.0. See our Documentation on version rules. Finally, I create a tag `example` just as an example.

```powershell
$newChannelRules.ActionPackages.Add("Deploy a Package")
$newChannelRules.VersionRange = "[1.0.0, 1.1.0]"
$newChannelRules.Tag = "example"
```

Now save our new version rule object to the Channel object.

```powershell
$newChannel.Rules = $newChannelrules
```

As we have in our previous scripts, we need to create the Channel by passing our `$newChannel` object into it.

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

Below is a list of common gotchas that I notice while working with the Octopus.Client. Hopefully this section will take some guess work out of issues you may encounter and allow you to avoid some common mistakes.

* When creating new objects, never provide an ID, Octopus will automatically fill the ID or create one where needed.

* Never specify a version of a resource. If you are modifying a variable of version 5, you do not need to tell Octopus that your modification is now version 6, Octopus will do this automatically. If you do specify this value, you run the risk of corrupting the resource by altering the version incorrectly or altering the version while another user is already editing the data. It's best to let Octopus handle ID's and Versions.

* When making a change via the API, it's important that you change the entire object. As an example, I'm not able to simply edit the script body and push the modified script body object to Octopus. I need to save the entire deployment process to Octopus. The same goes for modifying the Upgrade Locked on the Tentacle in my first example. I need to save the entire machine object after my changes, not just the nested object containing the changed value.

* Working with the API opens you up to potentially corrupting data if you are not conscious about the edits you make. 



### Conclusion

That concludes my introductory Octopus.Client API tutorial. This tutorial is intended for new users to the Octopus.Client / Octopus API. In future posts I will expand on the above concepts and provide examples for working on a larger scale and provide alternatives to using the Octopus.Client and PowerShell, including our SwaggerUI and CURL.

For further script examples, visit our [open source Octopus.Client script repository on GitHub](https://github.com/OctopusDeploy/OctopusDeploy-Api/tree/master/Octopus.Client).
