---
title: Request for Comments - Migrating from scriptcs to dotnet-script
description: Provide your feedback on the proposed migration.
author: isaac.calligeros@octopus.com
visibility: public
published: 2021-07-05-1400
metaImage: blogimage-feedback_2022-q3.png
bannerImage: blogimage-feedback_2022-q3.png
bannerImageAlt: Octopus salesperson at laptop with headset and icons representing customer feedback
isFeatured: false
tags:
 - Engineering
 - Product
---

We've received [user feedback](https://help.octopus.com/t/consider-use-dotnet-script-vs-scriptcs/22144) and [voting](https://octopusdeploy.uservoice.com/forums/170787-general/suggestions/31454668-allow-the-use-of-c-script-csx-using-net-core) requesting an update of the Calamari C# script executor from [scriptcs](https://github.com/scriptcs/scriptcs) to [dotnet-script](https://github.com/filipw/dotnet-script). This unlocks newer language features, referencing NuGet packages directly from within a script and removes our dependency on Mono for linux deployment targets.

This request for comments aims to gather feedback on the demand for this functionality and raise awareness of the tradeoffs in deprecating scriptcs. C# Scripting accounts for ~5% of our script steps and most instances of Calamari run on netcore 3.1 meaning this change is likely to only affect a small subset of customers.  The deployment targets affected by this change are linux targets using SSH with Mono enabled and Tentacles running on windows versions earlier than 2012R2. This is a breaking change for these customers running C# scripting and it's important to us to understand who is likely to be affected and help update these deployment processes. Any and all feedback would be appreciated.

## How we propose to support established dotnet-script

This Request for Comments (RFC) proposes removing scriptcs in favour of dotnet-script. In doing so C# scripting will no longer be available  on Linux deployment targets using SSH, this can be resolved by using the self-contained Calamari which runs on netcore3.1. (Note check this final statement)

Calamari currently supports .Net Framework 4.0.0, 4.5.2 and Core 3.1 while dotnet-script only supports Core 3.1 and above. This means C# scripting will also be unavailable to deployments working with Tentacles installed on versions of Windows earlier than 2012R2, these run the Framework builds of Calamari. 

### Added functionality
| Feature                               | scriptcs          | dotnet-script    |
|---------------------------------------|-------------------|------------------|
| C# Version                            | 5                 | 8                |
| Removes Mono Depdency                 | ❌                | ✅              |
| Nuget import support                  | ❌                | ✅              |
| Allows future .Net 5 & 6 support      | ❌                | ✅              |


## Benefits of the proposed approach

All C# language features included up to version [8](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-8) are now available for use in your C# scripts.

Removing the dependency on Mono to execute scripts brings us inline with modern cross platform .Net framework capabilities reducing the complexity of calling into Mono and related issues.

The Nuget import support from dotnet-script allows for the direct referencing of a nuget package in a script without having to [include the dll in the in the scripts packages](https://octopus.com/docs/octopus-rest-api/octopus.client/using-client-in-octopus). The new approach can be seen below.

```
#r "nuget: RestSharp, 108.0.1"

using RestSharp;
  
var client = new RestClient("https://pokeapi.co/api/v2/");
var request = new RestRequest("pokemon/ditto");
var response = await client.ExecuteGetAsync(request);
Console.WriteLine(response.Content);
```

dotnet-script supports .Net 5 and 6, this should help towards future upgrades of Calamari and allow the use of the latest language features in your scripts.

## When will this be released?

We are still considering how many existing deployment processes this change is likely to affect. The reliability of your deployments are our main priority and as such we don't have an immediate plan to release these changes.

## We want your feedback

We're still planning the second milestone, so now is a great time to help shape this new feature with your feedback. We have created a [GitHub issue to capture the discussion](https://github.com/OctopusDeploy/StepsFeedback/issues/7).

Specifically, we want to know:

- Will the limitations of Linux SSH targets or Windows versions older than 2012R2 affect you?
- If so can you foresee any challenges that may stop you from upgrading these deployment targets or using alternative scripting languages?
- Do the newer language features, easier NuGet package reference, and added reliability in removing Mono justify these changes?

This feedback will help us deliver the best solution we can.

<span><a class="btn btn-success" href="https://github.com/OctopusDeploy/StepsFeedback/issues/7">Provide feedback</a></span>

## Conclusion

In summary, the migration from ScriptCS to dotnet script includes results in the following changes:

- C# scripting deprecated for Linux SSH targets running Mono
- C# scripting deprecated for Windows deployment targets running on versions earlier than 2012R2
- Support for language features up to C# 8 from C# 5
- Direct imports of Nuget packages in scripts
- Removal of Mono dependence for calling scripts

Thanks for reading this post. We hope you're as excited about the proposed migration to dotnet-script as we are.

Any [feedback](https://github.com/OctopusDeploy/StepsFeedback/issues/7) you have is greatly appreciated.

Happy deployments!