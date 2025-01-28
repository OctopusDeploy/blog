---
title: Deprecations coming in 2025
description: Octopus will do some spring cleaning in 2025 and drop support for outdated features. Learn which ones and why.
author: robert.erez@octopus.com
visibility: private
published: 2026-01-01-1400
metaImage: img-blog-deprecated-features-2025.png
bannerImage: img-blog-deprecated-features-2025.png
bannerImageAlt: Silhouette of a woman wearing headphones with a mic in an Octopus branded tee holding a tablet.
isFeatured: false
tags: 
- Product
---

As part of our goal to improve predictability, the first release of each year tends to include deprecations users may need to consider before upgrading.

This post summarizes some those changes to expect in Octopus Server 2025.1.

## Changes in 2025

### Clarification To our support policies
Octopus supports a range of Windows and [non-Windows](https://octopus.com/docs/infrastructure/deployment-targets/linux#supported-distributions) platforms for use as targets and workers.

 Previously, we tied this list of supported platforms to those which we explicitly ran our full suite of automated tests against for every build. While this approach does give some degree of confidence that workloads _will_ generally work on those operating systems, it does make that list a little arbitrary. 
 
 Moving forward we will instead describe our platform support in terms of those platforms that are supported by the underlying tooling used for Octopus task execution, as well as being supported by the vendors themselves. Since the execution tooling, [Calamari](https://octopus.com/docs/octopus-rest-api/calamari), is currently built on [.NET 6](https://github.com/dotnet/core/blob/main/release-notes/6.0/supported-os.md#linux), this defines for us the lower bound version of those platforms that we expect workloads be able to run on. Note that we do intend to upgrade to [.NET 8](https://github.com/dotnet/core/blob/main/release-notes/8.0/supported-os.md#linux) in the next several months which may have some impact on support for older Linux platforms, such as Debian 11.

#### What to expect
Based on the reported usage metrics, for the vast majority of our customers this will not impact their targets that are in use. Most customers are utilizing Operating Systems, either Windows or Linux, which continue to be well within the supported version range.

It is also possible for some older Linux platforms that are not listed as explicitly supported, that deployments continue to work. This change in our policy does not mean we will necessarily do anything to _explicitly prevent_ tasks from running, only that we can only reasonably support and consider targets and workers that all our relevant dependencies also support. 

In future as we update the tooling used, expect to see the details on this support page to continue to be updated. Rest assured however that our goal is to continue to provide as much coverage to platforms that are in use by our customers, while providing  improved predictability into what we support and why.

### Windows Server 2008 targets

The latest version of [Tentacle](https://octopus.com/docs/infrastructure/deployment-targets/tentacle/windows/requirements#windows-server) has a minimum Windows requirement of Server 2012, yet we have still been supporting workloads on older Tentacle agents running on Windows Server 2008. 

Microsoft [dropped extended support](https://learn.microsoft.com/en-us/lifecycle/products/windows-server-2008) for the Windows Server 2008 family in January 2020. This operating system is also the last Windows OS that does not support .NET Core, one of the languages used to build Octopus Deploy. The complexity required to support this legacy platform now outweighs the value to our customers.

As noted above, we are updating our support policy to only cover platforms supported by the vendors themselves, so from `2025.1` Octopus will no longer support running workloads on Windows machines running on any version of Windows Server 2008 or older. Users should expect that changes may be introduced at any point that prevent standard deployment and runbook tasks from executing on these operating system.

:::hint
Please note, there is no requirement changes for later Windows versions or Linux targets as part of this change since modern platforms instead utilize .NET Core.
:::

#### What to expect
As outlined in our documentation, [limited support](https://octopus.com/docs/infrastructure/deployment-targets/tentacle/windows#windows-server-2008-limited-support) will be available for a short period to allow for forcing Octopus to use a previously compatible version of the [Calamari](https://octopus.com/docs/octopus-rest-api/calamari) execution tool. This work around should be considered a temporary solution when an upgrade beyond Windows Server 2008 is not yet possible and instead customers should aim to upgrade their Windows machines as soon as possible.

If your organization is not yet ready for these changes to the supported Windows Operating Systems, we recommend that you delay your Octopus Server upgrade until there is no longer a reliance on this older platform.

### Bundled Tooling
When Octopus introduced first class support for Azure and Aws steps, some of the CLI tooling required to interact with these cloud vendors were bundled with Octopus Server. Although this was a convenient way to bootstrap the ability to perform these deployments with minimal configuration on the target, this mechanism has had many drawbacks.

Since the tool is bundled into the Octopus Server instance itself, not only does this increase the installation size dramatically, users have less control over what version of that tool is used during a deployment. This means that potentially outdated or vulnerable versions of that tool continue to be used by older versions of Octopus Server when newer versions are available. 

#### What to expect
From `2025.1`, Azure, Aws and Terraform bundled CLI tools will no longer be distributed with Octopus itself and instead deployments will rely on them being available on the target or worker.

If you’re currently using these bundled tools today, you will already be seeing warnings indicating their imminent removal. From `2025.1` however, targets that rely on these tools to be distributed during a deployment may result in failed deployments.

You will need to either ensure the required tools are pre-installed on your targets, or alternatively you make use of [execution containers](https://octopus.com/docs/projects/steps/execution-containers-for-workers) that execute the workload from inside a container.

### Helm v2
Helm Deployments in Octopus currently rely on the Helm v3 CLI. This means that the behaviour that you expect when running Helm locally, is the same behaviour that Octopus utilizes during your deployments. As Helm v2 was officially deprecated in [November 2020](https://helm.sh/blog/helm-v2-deprecation-timeline/) we have decided that attempting to provide fallback support for users opting into using the v2 executable is no longer providing value.

#### What to expect
Assuming that the Helm v3 executable is available, and likely already being used, nothing will change. Octopus will just invoke the Helm tool that it finds. 

If you had been relying on the Helm v2 executable for your deployments, there are some breaking changes that exist between it and the v3 version. The [official Helm V2 to V3 migration guide](https://helm.sh/docs/topics/v2_v3_migration/) provides detailed information on the migration to Helm v3. 

### AzureRM PowerShell
The AzureRM PowerShell modules were Microsoft’s original mechanism for interacting with Azure resources via PowerShell. Microsoft has since deprecated AzureRM in favour of the Azure [az cli](https://learn.microsoft.com/en-us/cli/azure/) and [PowerShell Modules](https://learn.microsoft.com/en-us/powershell/azure/new-azureps-module-az?view=azps-13.0.0)..

AzureRM was [deprecated by Microsoft](https://learn.microsoft.com/en-us/powershell/azure/azurerm-retirement-overview) as of February 29, 2024 so Octopus will follow by removing in-built support for AzureRM PowerShell modules from `2025.1`

#### What to expect
If you have any [Azure PowerShell Steps](https://octopus.com/docs/deployments/azure/running-azure-powershell) that rely on the AzureRM PowerShell modules being available, then these steps will need to be updated to use the alternative tools described above.

## Summary
Each year brings more and more capabilities to Octopus, providing further abilities to meet the demands of modern continuous delivery pipelines. As part of this growth, Octopus Deploy must continue to review those features it already has to ensure the product remains as efficient and risk free as possible.

Our goal is always to be transparent about the future of our product and help you as much as possible with changes and migrations.

If any of the updates mentioned in this post surprise or concern you, please contact our helpful [support team](mailto:support@octopus.com).

Happy deployments!