---
title: Deprecations coming in 2024
description: Octopus will use the first release of 2024 to do some spring cleaning and deprecate outdated features. Learn which ones and why.
author: robert.erez@octopus.com
visibility: public
published: 2023-12-04-1400
metaImage: img-blog-unsupporteddeploymenttargetincwindows-2023.png
bannerImage: img-blog-unsupporteddeploymenttargetincwindows-2023.png
bannerImageAlt: Silhouette of a woman wearing headphones with a mic in an Octopus branded tee holding a tablet.
tags: 
- Product
- Azure
---

In the Octopus Server 2024.1 release, we'll make changes to modernize the feature set by beginning to drop support for some heritage features.

This post summarizes changes in future releases of Octopus Server.

## Deprecations and changes in 2024.1

### Windows Server 2003 targets

The latest version of [Tentacle](https://octopus.com/docs/infrastructure/deployment-targets/tentacle/windows/requirements#windows-server) has a minimum Windows requirement of Server 2012. But, we still support workloads to older Tentacle agents running on Windows Server 2003. To modernize our execution engine, we're beginning to only support platforms supported by the vendors themselves. 

Our first step is dropping support for running workloads on Windows Server 2003 targets and Workers.

For Windows Machines running Windows Server 2012 and below, this change means the minimum requirement is that you have .NET 462 installed. 

:::hint
Please note, this is not a requirement for later Windows versions or Linux targets as modern platforms instead use .NET Core.
:::

#### What to expect

As we upgrade our tooling to use .NET462 for targeting heritage operating systems, machines running Windows Server 2012 and earlier won't be able to run standard workloads unless they have installed the .NET462 framework or later. Health checks will provide a warning if Octopus detects this scenario. Due to the inability to run .NET462 on Windows Server 2003, these targets won't be able to run any standard Octopus steps unless the process uses [raw scripting](https://octopus.com/docs/deployments/custom-scripts/raw-scripting).

For more background on this change, please read our post, [Dropping support for Windows Server 2003 machines](https://octopus.com/blog/deprecating-win2003).

### Mono

The Mono runtime allowed Octopus to deploy to Linux servers before .NET Core was available. It also enabled advanced deployments of thousands of releases that wouldn't have been possible otherwise. 

Since then, .NET Core has become the standard execution framework for .NET workloads. And we've since provided both options for our customers depending on their target machine capabilities.

To simplify our codebase and improve stability, we've had to consider the trade-offs in supporting both Mono and .NET Core for Linux targets. It now makes more sense to focus on .NET Core support. Its cross-platform capabilities mean we can build and orchestrate the same tools for Linux or Windows and ultimately .NET Core, the future of .NET development. 

If you're running Mono on your Linux targets today, the migration process is likely to be as simple as checking a radio box on your SSH target in Octopus. There are also some [small caveats](https://octopus.com/blog/deprecating-mono#impacts) about the platforms that we'll continue to support.

#### What to expect

Attempting to run a step using an SSH target set up to use the **Calamari on Mono** target runtime will fail to execute. The option, while still visible, will be read-only and you can't create further targets of this type.

For more information about this change, please read our post, [Deprecating Mono](https://octopus.com/blog/deprecating-mono).

### Azure Cloud Services

In 2013, Octopus provided built-in support for one of the first cloud products, [Azure Cloud Services](https://octopus.com/blog/octopus-azure-deployments). 

Since then, Azure has introduced dozens of new products and capabilities. And Octopus has often followed up with relevant built-in capabilities. 

However, Azure has announced the sunsetting of the original Cloud Service resource, renamed Cloud Services Classic. The [final retirement date is set as August 31, 2024](https://learn.microsoft.com/en-us/lifecycle/products/azure-cloud-services-classic). In around 6 months, teams still relying on this cloud service won't be able to deploy to it at all, with Octopus Deploy or otherwise.

Over the next 6 months, we'll gradually remove our support in the new versions of Octopus Server for:

- Azure Cloud Service target
- Azure Cloud Service step
- Management Certificate account type

#### Migration options

Azure recommends migrating to [Azure Cloud Services extended support](https://learn.microsoft.com/en-us/azure/cloud-services-extended-support/overview). This will let you continue using your existing application code with minimal changes, but has other implications for Octopus. Because of changes to the deployment model, the use, and likely temporary nature, Octopus is unlikely to support this like it did the Classic services.

#### What to expect
You'll no longer be able to create the Azure Cloud Service target, Azure Cloud Service step, and Management Certificates. They'll be read-only. If you use any of these, you'll see a warning in Octopus.

To learn more, read our original post about this change, [What does Microsoft deprecating Azure Service Management APIs mean for Octopus users?](https://octopus.com/blog/azure-management-certs)

### F sharp

We originally supported running scripts with F# because of customer requests. However, we never saw high adoption rates. 

The tool we use to invoke our customer's F# scripts isn't compatible with the .NET Core framework. Since we're moving our tooling towards .NET Core, and F# has low uptake, it makes sense to drop support for F#.

We recommend migrating any F# scripts to one of our existing [built-in scripting options](https://octopus.com/docs/deployments/custom-scripts). You can also bundle and package your scripts and invoke them directly on the target using something like the [F# interactive tool](https://learn.microsoft.com/en-us/dotnet/fsharp/language-reference/fsharp-interactive-options). You can embed this as an additional deployment package or pre-install it on your target. 

:::warning
It's important to note that continuing to rely on your F# scripts will require changes to the way they access Octopus variables. This is because none of the [utility methods](https://octopus.com/docs/deployments/custom-scripts/using-variables-in-scripts) will be automatically available.
:::

#### What to expect

If you use F# for [any task](https://octopus.com/docs/deployments/custom-scripts#how-to-use-custom-scripts), either as a script step or a pre/post deployment script, you'll see warnings added to the deployment task, and F# will get removed as a scripting option. By 2024.3 these tasks will fail immediately.

## Future deprecations

We're planning more updates soon that may result in breaking changes for some customers.

### Windows Server 2008

For some of the [same reasons](https://octopus.com/blog/deprecating-win2003) that we're dropping support for Windows Server 2003, we plan to drop support for Windows Server 2008 in the 2025.1 release in 12 months. 

Microsoft flagged [end of support](https://learn.microsoft.com/en-us/troubleshoot/windows-server/windows-server-eos-faq/end-of-support-windows-server-2008-2008r2) for the Windows Server 2008 family (Standard and R2) in January 2020. As mentioned, we aim to only support platforms supported by the platform vendors themselves. Unsupported platforms introduce risks to our customers with unpatched security vulnerabilities. Plus we want to encourage migration to more modern operating systems.

To give you as much lead time as possible, we'll introduce warnings in Octopus Server 2024.1 when a task is being run against a Windows Server 2008 machine.

Before any final removal of support, we'll provide details about the change and your migration options.

### ScriptCS

Earlier in 2023, we let you know we're moving the [C# script execution engine, from using ScriptCS to dotnet script](https://octopus.com/blog/rfc-migrate-scriptcs-dotnet-script). 

We're now midway through this process, so we temporarily support both mechanisms, controllable via a project variable. The behavior defaults to the existing ScriptCS library. This lets you opt in to the modern approach as it may need updates to some scripts as outlined in our [blog post](https://octopus.com/blog/rfc-migrate-scriptcs-dotnet-script). 

We aim to swap the default engine used by 2024.3 and then remove ScriptCS entirely by 2025.1.

### Octo CLI

While not directly tied to a specific Octopus Server version, we've [redesigned the CLI tool](https://octopus.com/blog/building-octopus-cli-vnext) used to interact with your Octopus Server. The new tool provides a richer and more interactive experience that helps you fall into the pit of success. 

Although  we're [no longer updating the old CLI](https://octopus.com/blog/deprecating-octo-cli), and we'll eventually remove it from our [Downloads page](https://octopus.com/downloads), we designed our API to remain backwards compatible. This means we don't expect the use of the old CLI to cause any problems for the foreseeable future.

## Summary

As Octopus works to make your modern deployment scenarios simple, we must continuously consider the features and platforms we support. Sometimes the use of a feature doesn't justify the time to maintain and test it. Other times, changes by third-party vendors make the decision for us. 

Our goal is always to be transparent about the future of our product and help you as much as possible with changes and migrations.

If any of the changes mentioned in this post surprise or concern you, please contact our helpful [support team](mailto:support@octopus.com).

Happy deployments!
