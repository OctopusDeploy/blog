---
title: Request for Comments - Migrating from scriptcs to dotnet-script
description: Provide your feedback on the proposed migration from scriptcs to dotnet-script.
author: isaac.calligeros@octopus.com
visibility: public
published: 2022-07-20-1400
metaImage: blogimage-feedback_2021_01.png
bannerImage: blogimage-feedback_2021_01.png
bannerImageAlt: Octopus salesperson at laptop with headset and icons representing customer feedback
isFeatured: false
tags:
 - Product
---

:::warning
We shipped this change as a part of 2023.4. ScriptCS is still included as the default C# scripting engine with the option to use dotnet-script available with a project-specific variable `Octopus.Action.Script.CSharp.UseDotnetScript` and an environment-wide feature toggle `OCTOPUS__FeatureToggles__UseDotnetScriptCSharpExecutorFeatureToggle`. Please contact [support](mailto:support@octopus.com) if you're experiencing issues or would like dotnet-script enabled.
For further details on upgrading to dotnet-script please see the migration section.
:::

We received [customer feedback](https://help.octopus.com/t/consider-use-dotnet-script-vs-scriptcs/22144) and [UserVoice voting](https://octopusdeploy.uservoice.com/forums/170787-general/suggestions/31454668-allow-the-use-of-c-script-csx-using-net-core) requesting we update the tooling Octopus uses to run C# scripts, from [scriptcs](https://github.com/scriptcs/scriptcs) to [dotnet-script](https://github.com/filipw/dotnet-script). This would: 

- Unlock newer C# language features in deployment scripts
- Allow referencing NuGet packages directly from within scripts
- Remove the need to have Mono installed to run C# scripts on Linux deployment targets

C# scripting accounts for ~5% of our script steps, so we want to understand the impact this change could have on our users.

If you're using C# scripts in your deployment processes, and are deploying to Linux targets using SSH and Mono, or to Windows Tentacle targets running Windows versions earlier than 2012 R2, the proposed changes could impact you.

This post outlines the potential changes, plus the trade-offs in moving to dotnet-script and deprecating scriptcs. We also created a [GitHub issue](https://github.com/OctopusDeploy/StepsFeedback/issues/9) where you can provide feedback, and we can further gauge the demand for this functionality.

## How we propose to support dotnet-script

This Request for Comments (RFC) proposes removing `scriptcs` in favor of `dotnet-script`.

To deploy software to your server we use [Tentacle](https://github.com/OctopusDeploy/OctopusTentacle), a lightweight service responsible for communicating with Octopus Server, and invoking [Calamari](https://github.com/OctopusDeploy/Calamari). Calamari is a command-line tool that knows how to perform the deployment, and is the host process for all deployment actions including script execution. We currently build Calamari for .NET Framework 4.0.0, 4.5.2, and net6.0. Depending on your server OS, architecture and version, Tentacle receives one of these Calamari builds.

Historically, Calamari required Mono to be installed on your Linux targets to execute `scriptcs` as it's built on the full .NET Framework. With the introduction of cross-platform .NET apps with netcore3.1 Linux can now natively run .NET apps removing the complexity and overhead of Mono. Linux targets currently receive the net6.0 Calamari by default, with the exception of [Linux SSH targets](https://octopus.com/docs/infrastructure/deployment-targets/linux/ssh-target#add-an-ssh-connection), which can specify to run scripts on Mono.

`dotnet-script` is a modern implementation of C# scripting, built on .NET. It can run on all targets that support .NET apps (net6.0 and newer). If we make this change, it would mean that C# scripts would only be able to be run on targets that support .NET. Windows Server 2012 R2 and earlier only support .NET Framework, so these targets would lose the ability to run C# scripts.

## Impacts

### Added functionality
| Feature                               | scriptcs          | dotnet-script    |
|---------------------------------------|-------------------|------------------|
| C# version                            | 5                 | 8                |
| Removes Mono dependency for Linux      | ❌                | ✅              |
| NuGet import support                  | ❌                | ✅              |
| Allows future .NET 5 & 6 support      | ❌                | ✅              |

### Benefits of the proposed approach

All C# language features included up to [version 8](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-8) are now available for use in your C# scripts.

Removing the dependency on Mono to execute scripts brings us inline with modern cross platform .NET capabilities reducing the complexity of calling into Mono and related issues.

The NuGet import support from `dotnet-script` allows for the direct referencing of a NuGet package in a script without having to [include the dll in the scripts packages](https://octopus.com/docs/octopus-rest-api/octopus.client/using-client-in-octopus). The new approach can be seen below.

```
#r "nuget: RestSharp, 108.0.1"

using RestSharp;
  
var client = new RestClient("https://pokeapi.co/api/v2/");
var request = new RestRequest("pokemon/ditto");
var response = await client.ExecuteGetAsync(request);
Console.WriteLine(response.Content);
```

### Linux SSH Targets using Mono
One trade-off of this change is that C# scripting would no longer be available on Linux deployment targets using SSH with Mono.

#### Migration

This migration to dotnet-script introduces a dependency on net6.0, any Workers or deployment targets running C# scripts using dotnet-script will require the net6.0 SDK on path.

The Octopus class has also been removed from C# scripting, this brings behaviour inline with PowerShell scripts. This changes behaviour like setting variables from `Octopus.SetVariable` to `SetVariable`. Parameters have also been changed from `Octopus.Parameters` to `OctopusParameters`, for a full list of these methods see the [bootstrap code](https://github.com/OctopusDeploy/Calamari/blob/master/source/Calamari.Common/Features/Scripting/DotnetScript/Bootstrap.csx). The old bootstrapper is still available by setting the `Octopus.Action.Script.CSharp.UseOctopusClassBootstrapper` variable, however this is only intended for migration purposes and is due for deprecation in the near future.

To run C# scripts against your SSH linux targets, you'd need to reconfigure your SSH targets to use the self-contained Calamari which runs via net6.0. This requires the net6.0 SDK on the machine executing dotnet-script.

To do this, [select the Self-Contained Calamari target runtime on your SSH target](https://octopus.com/docs/infrastructure/deployment-targets/linux/ssh-target#self-contained-calamari). Targets using the Linux tentacle will continue to work as they always have.

### Windows Server 2012 R2 (and earlier) targets
The other trade-off with this change is that `dotnet-script` only works with net6.0 and above. This would make C# scripting unavailable to deployments against Windows Tentacles installed on versions of Windows earlier than 2012 R2, as these run .NET Framework builds of Calamari. 

#### Workaround

We developed a workaround so you can continue using scriptcs on your affected Windows targets, but you'll have to update your deployment process. 

1. Add the [scriptcs NuGet package](https://www.nuget.org/packages/scriptcs/) as a [referenced package](https://octopus.com/blog/script-step-packages).
2. Copy the body of your C# Script into the `$ScriptContent` variable in the PowerShell template below.

:::info
Any paramaters used in the C# script need to be passed in through the scriptcs arguments and referenced using the `Env.ScriptArgs[Index]` format inside the ScriptContent. The template below shows an example of how to do this for `Octopus.Deployment.Id`.
:::

##### Powershell
```powershell
$ScriptContent = @"
Console.WriteLine(Env.ScriptArgs[0]);
"@

New-Item -Path . -Name "ScriptFile.csx" -ItemType "file" -Value $ScriptContent

$scriptCs = Join-Path $OctopusParameters["Octopus.Action.Package[scriptcs].ExtractedPath"] "tools/scriptcs.exe"

& $scriptCs ScriptFile.csx -- $OctopusParameters["Octopus.Deployment.Id"]
```

## When will this be released?

We're still evaluating how many users this change is likely to affect. We won't make or release the proposed changes until we have a clear picture of who we'll impact and what actions they'd need to take.

## We want your feedback

We're still considering this change, so now is a great time to help shape this proposal with your feedback. We created a [GitHub issue to capture the discussion](https://github.com/OctopusDeploy/StepsFeedback/issues/9).

Specifically, we want to know:

- Will the limitations of Linux SSH targets or Windows versions older than 2012 R2 affect you?
- If so, can you foresee any challenges that may stop you from upgrading these deployment targets or using alternative scripting languages?
- Do the newer language features, easier NuGet package reference, and added reliability in removing Mono justify these changes?

Your feedback will help us deliver the best solution we can.

<span><a class="btn btn-success" href="https://github.com/OctopusDeploy/StepsFeedback/issues/9">Provide feedback</a></span>

## Conclusion

In summary, the migration from `scriptcs` to `dotnet-script` will result in the following changes:

- C# scripting deprecated for Linux SSH targets running Mono
- C# scripting deprecated for Windows deployment targets running on versions earlier than 2012 R2
- Increase support for language features from C# 5 to C# 8
- Direct imports of NuGet packages in scripts
- Removal of Mono requirement for running C# scripts on Linux targets

Thanks for reading this RFC. Any [feedback](https://github.com/OctopusDeploy/StepsFeedback/issues/9) you have is greatly appreciated.

Happy deployments!
