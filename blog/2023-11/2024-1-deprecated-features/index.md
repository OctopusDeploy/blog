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

The upcoming Octopus Server `2024.1` release is shaping up to contain a series of changes to modernize the available feature set by deprecating support for some older legacy style features.

This post summarizes what changes to expect in future releases of Octopus Server.

## Deprecations and changes in 2024.1

### Windows Server 2003 Targets
Although the latest version of [Tentacle](https://octopus.com/docs/infrastructure/deployment-targets/tentacle/windows/requirements#windows-server) has a minimum Windows requirement of Server 2012, we surprisingly still support workloads to legacy Tentacle agents running on Windows Server 2003. In order to modernise our execution engine we are beginning to realign our supported platforms to those supported by the vendor themselves. 

Our first step in that direction is the dropping of the support for running workloads on Windows Server 2003 Targets and Workers.

For Windows Machines running Windows Server 2012 and below, this change upgrades the minimum requirement for .NET 462 to be installed. Note that this is not a requirement for later Windows versions or Linux targets as modern platforms instead utilize .NET Core.

#### What to expect
As we upgrade our tooling to use .NET462 for targeting legacy Operating Systems, machines running Windows Server 2012 and earlier will be unable to run standard workloads unless they have installed the .NET462 Framework or later. Health checks will provide a warning if this scenario is detected. Due to the inability to run .NET462 on Windows Server 2003, these targets will effectively be unable to run any standard Octopus steps unless the process happens to be using [Raw Scripting](https://octopus.com/docs/deployments/custom-scripts/raw-scripting).

Read the [blog post here](https://octopus.com/blog/deprecating-win2003) for more background on this change.

### Mono
The Mono runtime allowed Octopus to deploy to Linux servers before .NET Core was available and has enabled advanced deployments of many thousands of releases that would have otherwise not been possible. Since that time .NET Core has become the standard execution framework for .NET workloads and we have since provided both options for our customers depending on their target machine capabilities.

To simplify our codebase and improve stability, the trade off in supporting both Mono and .NET Core for Linux targets has now tilted more heavily toward focusing on just investing in .NET Core support. It's cross platform capabilities means that we can build and orchestrate the same tools for either Linux or Windows and ultimately .NET Core the future of .NET development. 

If you are running mono on your Linux targets today, the migration process is likely to be nothing more involved than updating a radio-box on your SSH Target in Octopus Deploy, with some [slight caveats](https://octopus.com/blog/deprecating-mono#impacts) in regards to the platforms that will continue to be supported.

#### What to expect
Attempting to run a step using an SSH Target that has been configured to use the `Calamari on Mono` Target Runtime will fail to execute. The option, while still visible will be read-only and no further targets of this type will be able to be created.

Read the [blog post here](https://octopus.com/blog/deprecating-mono) for more background on this change.

### Azure Cloud Services
One of the first cloud products provided that Octopus Deploy provided first class support for, was with [Azure's Cloud Services in 2018](https://octopus.com/blog/octopus-azure-deployments). 

Since that time, Azure have introduced dozens of new products and capabilities, many of which Octopus followed up with relevant built-in capabilities. In the meantime however, Azure have announced the sunsetting of the original Cloud Service resource, renamed Cloud Services Classic, with the [final retirement date set as August 31, 2024](https://learn.microsoft.com/en-us/lifecycle/products/azure-cloud-services-classic). In a little over 6 months teams, that are still relying on this cloud service will be unable to deploy to it at all, with Octopus Deploy or otherwise.

In preparation for this, over the next six months we will begin to gradually deprecate our support for Azure Cloud Service Target, Azure Cloud Service Step or Management Certificate account type in new versions of Octopus Server.

#### Migration options
The recommended approach by Azure is to migrate to using [Azure Cloud Services Extended Support](https://learn.microsoft.com/en-us/azure/cloud-services-extended-support/overview). While this will allow you to continue using your existing application code with minimal changes, the changes require to the deployment model and based on the usage and likely temporary nature means that Octopus is unlikely to support it with the same first-class level that it did for the Classic services. 

#### What to expect
The Azure Cloud Service Target, Azure Cloud Service Step and Management Certificates will no longer be able to be created and will be made read only. Any usage of these will result in in-app warnings.

Read the original [blog post here](https://octopus.com/blog/azure-management-certs) for background on this change.

### F#
Running scripts with F# was originally added partly due to a large number of customer requests at the time. The reality is that as with F# itself, the hype never converted into adoption. 

The tool through which we invoke our customer's F# scripts is not compatible with the .NET Core framework and since we are moving our tooling towards .NET Core, deprecating F# is the most practical option when considered alongside it's low uptake.

We recommend migrating any F# scripts to either one of our existing [built-in scripting options](https://octopus.com/docs/deployments/custom-scripts), or bundle and package up your scripts and invoke them directly on the target using something like the [F# interactive tool](https://learn.microsoft.com/en-us/dotnet/fsharp/language-reference/fsharp-interactive-options) either embedded as an additional deployment package or pre-installed on your target. It is important to note that continuing to rely on your own F# scripts will require some changes to the way that they access Octopus variables as none of the [utility methods](https://octopus.com/docs/deployments/custom-scripts/using-variables-in-scripts) will be automatically available.

#### What to expect
[Any task](https://octopus.com/docs/deployments/custom-scripts#how-to-use-custom-scripts) that utilizes F#, either as a script step or a pre/post deployment script will result in warnings added to the deployment task and F# will be removed as a scripting option. By `2024.3` these tasks will instead fail immediately.

## Future Deprecations
Additional updates are planned in the near term future that may result in breaking changes for some users.

#### Windows Server 2008
For some of the [same reasons](https://octopus.com/blog/deprecating-win2003) that we are deprecating support for Windows Server 2003, our plans are to drop support for Windows Server 2008 in the `2025.1` release in 12 months time. 

The extended support for the Windows Server 2008 family (Standard and R2) were flagged as End of Life by [Microsoft in January 2020](https://learn.microsoft.com/en-us/troubleshoot/windows-server/windows-server-eos-faq/end-of-support-windows-server-2008-2008r2) and we intend to align our supported platform list with those that are supported by the OS vendors themselves. Remaining on unsupported platforms introduces risks to our customers in terms on unpatched security vulnerabilities and we want to encourage the migration to more modern operating systems.

To provide as much lead time as possible for customers, we will start to introduce warnings in Octopus Server `2024.1` when a task is being run against a Windows Server 2008 machine.

Rest assured that before any final removal of support, we will provide more details about this change and any other migration options available.

#### ScriptCS
As noted [earlier this year](https://octopus.com/blog/rfc-migrate-scriptcs-dotnet-script), we are moving the C# script execution engine from using `ScriptCS` to `dotnet script`. 

We are currently midway through this process so temporarily support both mechanisms, controllable via a project variable. The behaviour currently defaults to the existing `ScriptCS` library to provide an opportunity for customers to opt into the modern approach as it may require updates to some scripts as outlined in the blog post. Our goal is to make the swap the default engine used by `2024.3` and then remove ScriptCS entirely by `2025.1`.

#### OctoCLI
While not directly tied to a specific Octopus Server version, we have [redesigned the CLI tool](https://octopus.com/blog/building-octopus-cli-vnext) used to interact with your Octopus Server. This redesigned tool provides a much richer and interactive experience that helps users fall into the pit of success. 

Although the legacy cli tool is no longer being updated and will eventually be removed from the download page, we design our API to remain backwards compatible as possible and so do not expect any usages of the existing cli tool to run into any problems in the foreseeable future.

## Summary
As Octopus continues support the modern deployment scenarios that our customers are facing, we must continuously take stock of the features and platforms that we support. Sometimes the usage of a feature does not justify the maintenance burden in continuing to maintain and test it, and other times our hand is dealt for us by changes to third party vendors. 

In all these cases, our goal is to be transparent about the future of our product and provide as much assistance as possible to our users through the relevant migration process.

If any of these changes surprises or concerns you, please get in contact with our helpful support staff for more information.

Happy Deployments!
