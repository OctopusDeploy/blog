---
title: Deprecations Coming in 2024.1
description: Octopus Deploy will be using the first release of 2024 to perform some spring cleaning and deprecate some outdated features
author: robert.erez@octopus.com
visibility: public
published: 2026-06-05-1400
metaImage: 
bannerImage: 
bannerImageAlt: 
isFeatured: false
tags: 
- Product
- Azure
---

The upcoming Octopus Server `2024.1` release is shaping up to contain a series of changes to modernize the available featureset by deprecating support for some older legacy style features.

This post summarizes what changes to expect in future releases of Octopus Server.

## Windows Server 2003 Targets
Although the latest version of [Tentacle](https://octopus.com/docs/infrastructure/deployment-targets/tentacle/windows/requirements#windows-server) has a minumum Windows requirement of Server 2012, we suprisingly still support workloads to legacy Tentacle agents running on Windows Server 2003. In order to modernise our execution engine we are beginning to realign our supported platforms to those supported by the vendor themselves. 

Our first step in that direction is the dropping of the support for running workloads on Windows Server 2003 Targets and Workers.

For Windows Machines running Windows Server 2012 and below, this change upgrades the minimum requirement for .NET 462 to be installed. Note that this is not a requirement for later Windows versions or Linux targets as modern platforms instead utilize .NET Core.

Read the [blog post here](https://octopus.com/blog/deprecating-win2003) for more background on this change.

## Mono
The Mono runtime allowed Octopus to deploy to Linux servers before .NET Core was available and has enabled advanced deployments of many thousands of releases that would have otherwise not been possible. Since that time .NET Core has become the standard execution framework for .NET workloads and we have since provided both options for our customers depending on their target machine capabilities.

To simplify our codebase and improve stability, the trade off in supporting both Mono and .NET Core for Linux targets has now tilted more heavily toward focusing on just investing in .NET Core support. It's cross platform capabilties means that we can build and orchestrate the same tools for either Linux or Windows and ultimately .NET Core the future of .NET development. 

If you are running mono on your Linux targets today, the migration process is likely to be nothing more involved than updating a radio-box on your SSH Target in Octopus Dpeloy, with some [slight caveats](https://octopus.com/blog/deprecating-mono#impacts) in regards to the platforms that will continue to be supported.

Read the [blog post here](https://octopus.com/blog/deprecating-mono) for more background on this change.

## Azure Cloud Services
One of the first cloud products provided that Octopus Deploy provided first class support for, was with [Azure's Cloud Services in 2018](https://octopus.com/blog/octopus-azure-deployments). 

Since that time, Azure have introduced dozens of new products and capabilities, many of which Octopus followed up with relevant built-in capabilities. In the meantime however, Azure have announced the sunsetting of the original Cloud Service resource, renamed Cloud Services Classic, with the [final retirement date set as August 31, 2024](https://learn.microsoft.com/en-us/lifecycle/products/azure-cloud-services-classic). In a little over 6 months teams, that are still relying on this cloud service will be unable to deploy to it at all, with Octopus Deploy or otherwise.

In preparation for this, over the next six months we will begin to gradually deprecate our support for Azure Cloud Servie Target, Azure Cloud Service Step or Management Certificate account type in new versions of Octopus Server. These Management Certificates have since bee repleased by Azure by the Resource Service authentication model.

### Migration
The reccomended approach by Azure is to migrate to using [Azure Cloud Services Extended Support](https://learn.microsoft.com/en-us/azure/cloud-services-extended-support/overview). While this will allow you to continue using your existing application code with minimal changes, the changes require to the deployment model and based on the usage and likely temporary nature means that Octopus is unlikely to support it with the same first-class level that it did for the Clasic services. 

Read the original [blog post here](https://octopus.com/blog/azure-management-certs) for background on this change.

## F#
Running scripts with F# was originally added partly due to a large number of customer requests at the time. The reality is that as with F# itself, the hype never converted into adoption. 

The tool through which we invoke our customer's F# scripts is not compatable with the .NET Core framework and since we are moving our tooling towards .NET Core, deprecating F# is the most practical option when considered alongside it's low uptake.

We reccomend migrating any F# scripts to either one of our existing [built-in scripting options](https://octopus.com/docs/deployments/custom-scripts), or bundle and package up your scripts and invoke them directly on the target using something like the [F# interactive tool](https://learn.microsoft.com/en-us/dotnet/fsharp/language-reference/fsharp-interactive-options) either embeded as an addtional deployment package or pre-installed on your target. It is important to note that continuing to rely on your own F# scripts will require some changes to the way that they access Octopus variables as none of the [utiity methods](https://octopus.com/docs/deployments/custom-scripts/using-variables-in-scripts) will be automatically available.

## Future Deprecations
* Windows Server 2008 is in our sights for deprecation in 2025. This year we will begin introducing some early in-app warnings to provid customers ample warning.
* ScriptCS
* OctoCLI
