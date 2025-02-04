---
title: Deprecations coming in 2025
description: Octopus will do some spring cleaning in 2025 and drop support for outdated features. Learn which ones and why.
author: robert.erez@octopus.com
visibility: public
published: 2025-02-05-1400
metaImage: img-blog-unsupporteddeploymenttargetincwindows-2023.png
bannerImage: img-blog-unsupporteddeploymenttargetincwindows-2023.png
bannerImageAlt: Silhouette of a woman wearing headphones with a mic in an Octopus branded tee holding a tablet.
isFeatured: false
tags: 
- Product
---

As part of our goal to improve predictability, the first release of each year tends to include deprecations users may need to consider before upgrading.

This post summarizes some of the changes to expect in Octopus Server 2025.1.

## Changes in 2025

### Clarifying our support policies

Octopus supports a range of Windows and [non-Windows](https://octopus.com/docs/infrastructure/deployment-targets/linux#supported-distributions) platforms for use as targets and workers.

Previously, we tied our list of supported platforms to those we explicitly ran against our full suite of automated tests on every build. While this gives confidence that workloads *will* mostly work on those operating systems, it makes the list somewhat arbitrary. 
 
Moving forward, we'll instead describe our platform support as platforms that are supported by the underlying tooling used for Octopus task execution, and supported by the vendors themselves. 

Since the execution tooling, [Calamari](https://octopus.com/docs/octopus-rest-api/calamari), is built on [.NET 6](https://github.com/dotnet/core/blob/main/release-notes/6.0/supported-os.md#linux), this defines the lower bound version of the platforms we expect workloads be able to run on. 

:::warning
We intend to upgrade to [.NET 8](https://github.com/dotnet/core/blob/main/release-notes/8.0/supported-os.md#linux) in the next few months, and this may impact support for older Linux platforms, like Debian 11. 
:::

#### What to expect

Based on the usage metrics, this change to our support won't impact targets in use for the vast majority of our customers. Most customers use Windows or Linux operating systems that are well within the supported version range.

Deployments may continue to work for some older Linux platforms which aren't explicitly listed. The change in our policy doesn't mean we'll necessarily do anything to specifically prevent tasks from running. It instead means we can only reasonably support and consider targets and workers that all our relevant dependencies also support. 

In future, as we update the tooling used, we'll continue to share details on this support page. Our goal is to continue providing as much coverage to the platforms our customers use, while improving predictability about what we support and why.

### Windows Server 2008 targets

The latest version of [Tentacle](https://octopus.com/docs/infrastructure/deployment-targets/tentacle/windows/requirements#windows-server) has a minimum Windows requirement of Server 2012. However, we've still supported workloads on older Tentacle agents running on Windows Server 2008. 

Microsoft [dropped extended support](https://learn.microsoft.com/en-us/lifecycle/products/windows-server-2008) for the Windows Server 2008 family in January 2020. This operating system is also the last Windows OS that doesn't support .NET Core, one of the languages used to build Octopus Deploy. The complexity to support this legacy platform now outweighs the value to our customers.

As mentioned, we're updating our support policy to only cover platforms supported by the vendors themselves. As such, from **2025.1**, Octopus will no longer support running workloads on Windows machines running on any version of Windows Server 2008 or older. You can expect that we'll introduce changes in an upcoming release that prevents standard deployment and runbook tasks from executing on these operating systems.

:::hint
Please note, requirements don't change for later Windows versions or Linux targets as part of this update, since modern platforms instead use .NET Core.
:::

#### What to expect

As outlined in our docs, we'll offer [limited support](https://octopus.com/docs/infrastructure/deployment-targets/tentacle/windows#windows-server-2008-limited-support) for a short period so you can force Octopus to use a previously compatible version of the [Calamari](https://octopus.com/docs/octopus-rest-api/calamari) execution tool. This work around is only a temporary solution when an upgrade beyond Windows Server 2008 is not yet possible. Customers should aim to upgrade their Windows machines as soon as possible.

If your organization isn't ready for these changes to the supported Windows operating systems, we recommend delaying your Octopus Server upgrade until you no longer rely on this older platform.


### Bundled tooling

When Octopus introduced in-built support for Azure and AWS steps, some of the CLI tooling required to interact with these cloud vendors were bundled with Octopus Server. This was a convenient way to bootstrap the ability to perform these deployments with minimal configuration on the target. However, this mechanism has had many drawbacks.

Since the tool gets bundled into the Octopus Server instance itself, it increases the installation size dramatically and users have less control over what version of that tool gets used during a deployment. This means older versions of Octopus Server potentially use outdated or vulnerable versions of that tool when newer ones are available. 

#### What to expect

From **2025.1**,  Azure, AWS, and Terraform bundled CLI tools will no longer be distributed with Octopus itself. Instead, deployments will rely on them being available on the target or worker.

If you’re using these bundled tools today, you'll already see warnings about their imminent removal. In an upcoming release, targets that rely on these tools to be distributed during a deployment may result in failed deployments.

You need to either: 

- Make sure the required tools are pre-installed on your targets.
- Use [execution containers](https://octopus.com/docs/projects/steps/execution-containers-for-workers) that run the workload from inside a container that itself contains all the dependencies.

### Helm v2

Helm deployments in Octopus currently rely on the Helm v3 CLI. This means  the behavior you expect when running Helm locally is the same behaviour that Octopus uses during your deployments. 

As Helm v2 was officially deprecated in [November 2020](https://helm.sh/blog/helm-v2-deprecation-timeline/), we decided that trying to provide fallback support for customers using the v2 executable no longer delivers value.

#### What to expect

If the Helm v3 executable is available, and you're already using it, nothing will change. Octopus will just invoke the Helm tool that it finds.

If you rely on the Helm v2 executable for your deployments, there are some breaking changes that exist between the v2 and v3 versions. The [official Helm v2 to v3 migration guide](https://helm.sh/docs/topics/v2_v3_migration/) has detailed information on the migration to Helm v3. 

### Azure Resource Manager PowerShell

The Azure Resource Manager (AzureRM) PowerShell modules were Microsoft’s original way to interact with Azure resources via PowerShell. Microsoft has since deprecated AzureRM in favor of the [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/) and [PowerShell modules](https://learn.microsoft.com/en-us/powershell/azure/new-azureps-module-az?view=azps-13.0.0).

Microsoft [deprecated AzureRM](https://learn.microsoft.com/en-us/powershell/azure/azurerm-retirement-overview) as of February 29, 2024. Octopus will follow by removing in-built support for AzureRM PowerShell modules from **2025.1**.

#### What to expect

If you have any [Azure PowerShell steps](https://octopus.com/docs/deployments/azure/running-azure-powershell) that rely on the AzureRM PowerShell modules being available, you need to update these steps to use the alternative tools described above.

## Summary

Each year Octopus introduces more and more capabilities, to further meet the demands of modern Continuous Delivery pipelines. As part of this growth, Octopus must continue to review features to ensure the product remains as effective and risk-free as possible.

Our goal is always to be transparent about the future of our product and help you as much as we can with changes and migrations.

If any of the updates mentioned in this post surprise or concern you, please contact our [helpful support team](mailto:support@octopus.com).

Happy deployments!