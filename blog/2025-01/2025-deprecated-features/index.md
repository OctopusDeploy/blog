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

As part of our goal to improve predictability, the first release of each year tends to include significant deprecations users may need to consider before upgrading.

This post summarizes some those changes to expect in Octopus Server 2025.1.

## Deprecations and changes in 2025

### Windows Server 2008 targets

The latest version of [Tentacle](https://octopus.com/docs/infrastructure/deployment-targets/tentacle/windows/requirements#windows-server) has a minimum Windows requirement of Server 2012. Yet, we have still been supporting workloads on older Tentacle agents running on Windows Server 2008. To modernize our execution engine, we will shortly be updating our support polcy to only cover platforms supported by the vendors themselves. 

Microsoft [dropped extended support](https://learn.microsoft.com/en-us/lifecycle/products/windows-server-2008) for the Windows Server 2008 family in January 2020. This operating system is also the last Windows OS that does not support .NET Core, one of the languages used to build Octopus Deploy. The complexity required to support this legacy platform now outweighs the value to our customers.

From `2025.1`, Octopus will no longer support running workloads on Windows machines running on any version of Windows Server 2008 or older. Users should expect that changes may be introduced at any point that prevent standard deployment and runbook tasks from executing on these operating system.

:::hint
Please note, there is no requirement changes for later Windows versions or Linux targets as modern platforms instead use .NET Core.
:::

#### What to expect
As outlined in our documentation, [limited support](https://octopus.com/docs/infrastructure/deployment-targets/tentacle/windows#windows-server-2008-limited-support) will be available for a short period to allow forcing Octopus to use a previously compatable version of the [Calamari](https://octopus.com/docs/octopus-rest-api/calamari) execution tool. This work around should be considered a temporary solution when an upgrade beyond Windows Server 2008 is not yet possible and instead customers should aim to upgrade their Windows machines as soon as possible.

If your organization is not yet ready for these changes to the supported Windows Operating Systems, we reccomend that you delay your Octopus Server upgrade until there is no longer a reliance on this older platform.

### Bundled Tooling
When Octopus introduced first class support for Azure and Aws steps, some of the tooling required to interact with these cloud vendors were bundled with Octopus Server. Although this was a convenient way to bootstrap the ability to perform these deployments with minimal configuration on the target, this mechanism has had many drawbacks.

Since the tool is bundled into the Octopus Server instance itself, increasing the installation size dramatically, users have less control over what version of that tool is used during a deployment. This means that potentially outdated or vulnerable versions of that tool continue to be used when newer versions are available. 

#### What to expect
From `2025.1`, the bundled vendor tools will no longer be distributed with Octopus itself and instead deployments will rely on them being available on the target or worker.

If you’re currently using these bundled tools today, you will already be seeing warnings indicating their immenent removal. From `2025.1` however, targets that rely on these tools to be distributed during a deployment may result in failed deployments.

You will then need to either manually install the required tools on your workers directly, ensuring that they are available from the consolve via standard path or alternatively you make use of [execution containers](https://octopus.com/docs/projects/steps/execution-containers-for-workers) that execute the workload from inside a container.

### Helm v2
Helm Deployments in Octopus currently rely on the Helm v3 CLI. This means that the behaviour that you understand and rely on when running Helm locally, are the same behaviours that Octopus utilizes during your deployments. As Helm v2 was officially deprecated in [November 2020](https://helm.sh/blog/helm-v2-deprecation-timeline/) we have decided that attempting to provide fallback support for users opting into using the v2 executable is no longer providing value.


#### What to expect
Assuming that the Helm v3 executable is available, and likely already being used, nothing will change. Octopus will just invoke the Helm tool that it finds. 

If you had been relying on the Helm v2 executable for your deployments, there are some breaking changes that exist between it and the v3 version. The [official Helm V2 to V3 migration guide](https://helm.sh/docs/topics/v2_v3_migration/) provides detailed information on the migration to Helm v3. 


### AzureRM Powershell
The AzureRM Powershell modules were Microsoft’s way of integrating Powershell with Azure resources. Microsoft has deprecated AzureRM in favor of the Azure [az cli](https://learn.microsoft.com/en-us/cli/azure/) and [PowerShell Modules](https://learn.microsoft.com/en-us/powershell/azure/new-azureps-module-az?view=azps-13.0.0)..

AzureRM was [deprecated by Microsoft](https://learn.microsoft.com/en-us/powershell/azure/azurerm-retirement-overview) as of February 29, 2024 so Octopus will follow by removing in-built support for AzureRM powershell modules from `2025.1`

#### What to expect
Any existing first-class Azure steps will continue to function as before unchanged as these do not rely on the AzureRM Powershell modules. 

If you have any [Azure Powershell Steps](https://octopus.com/docs/deployments/azure/running-azure-powershell) that rely on the AzureRM Powershell modules being available, then these steps will need to be updated to use the alternative approaches described above .


## Summary
Each year brings more and more capabilities to Octopus, providing further abilities to meet the demands of modern continous delivery pipelines. 

As this list of features continues to grow, some cater for use cases which are no longer relevant due to the continuing evolution of the rest of the industry. To keep Octopus a streamlined and efficient as possible, this means that 

Our goal is always to be transparent about the future of our product and help you as much as possible with changes and migrations.

If any of the updates mentioned in this post surprise or concern you, please contact our helpful [support team](mailto:support@octopus.com).

Happy deployments!