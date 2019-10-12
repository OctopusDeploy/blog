---
title: Octopus Deploy 2019.10 - Flexible Linux deployments, PowerShell Core support, Operations RunBooks EAP
description: Octopus 2019.10 introduces Tentacle for Linux for flexible Linux deployments, built-in PowerShell Core support, simpler build information and releas notes, and early access to RunBooks for operations teams.
author: rob.pearson@octopus.com
visibility: private
published: 2019-10-14
metaImage: 
bannerImage: 
tags:
 - Product
---

<iframe width="560" height="315" src="https://www.youtube.com/embed/TODO" frameborder="0" allowfullscreen></iframe>

// LEE: Do you think this works as an intro or should we break it up more? I want the key benefits line for Linux Tentacle but otherwise, I'm happy to jump to specific sections.

We're happy to ship Octopus Deploy 2019.9 as this a feature packed release with some great benefits for teams. Our headline feature is Tentacle for Linux which provides greater flexibility for teams deploying to Linux and unlocks the ability to deploy to servers without the need to open port 22 for SSH on your servers. This release also adds built-in support for PowerShell Core, improved build information and work item tracking, simpler package-based step templates, and early access to our Runbooks for operations teams feature. 

<h2>In this post</h2>

!toc

## Tentacle for Linux

TODO: Add image.

Octopus first introduced support for Linux deployments over secure shell (SSH) in version 3.0 and it's a popular option for teams. However, some companies operate in highly secure environments where it's not possible to open port 22 on production servers. An example of this is where web applications operate  in their own DMZs with no incoming connections permitted other than HTTPS for web traffic. 

Thankfully, this scenario is now possible with Tentacle for Linux. Our Tentacle agent is a lightweight service that enables secure communication between the Octopus Server and deployment targets in listening and polling modes. In polling mode, the targets themselves phone home and contact the Octopus Server to see if there is any deployment work to do and if there is, they execute it. 

Tentacle for Linux provides greater flexibility for teams deploying to Linux in highly secured environments.

## PowerShell Core support

TODO: Add image.

This release adds built-in PowerShell Core support enabling teams to write cross-platform scripts using Microsoft's actively maintained automation framework. Our PowerShell Core support 'just works' and it pairs very well with our expanded support for Linux deloyments. Many development and operations teams are comfortable writing PowerShell scripts and PowerShell Core enables people with PowerShell skills to leverage those skills to write rich scripts for Linux platforms. 

**Windows platforms**

Octopus will use PowerShell Core if it's available otherwise, it will use PowerShell (full framework) as this is bundled with all supported Windows Servers. 

NOTE: Octopus allows you to specify which framework you wish to use as well. 

**Linux platforms**

Octopus will automatically execute scripts with PowerShell Core if it's installed. 


[Learn More](TODO: find link)

## Improved build information and work item tracking

## Simpler package-based step templates




## Runbooks for operations teams





## Breaking Changes

This release includes a major bump of Azure PowerShell  modules to `6.8.1` to fix a [known issue](https://github.com/OctopusDeploy/Issues/issues/4574) with the previous `5.7.0` bundled version. Please see [Azure PowerShell release notes](https://docs.microsoft.com/en-us/powershell/azure/release-notes-azureps?view=azurermps-6.11.0) for more information.

## Upgrading

As usual [steps for upgrading Octopus Deploy](https://octopus.com/docs/administration/upgrading) apply. Please see the [release notes](https://octopus.com/downloads/compare?to=2018.9.0) for further information. Self-Hosted Octopus customers can [download](https://octopus.com/downloads/2018.9.0) the latest release now. For Octopus Cloud, you will start receiving the latest bits next week during their maintenance window.

## Wrap up

That's it for this release. Feel free to leave us a comment and let us know what you think! Happy deployments!