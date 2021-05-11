---
title: Exporting and Importing Projects between Spaces 
description: In Octopus 2021.1 we've introduced a new feature to export and import projects 
author: michael.richardson@octopus.com
visibility: private
published: 2222-01-01
tags:
 - Product 
---

**The 2021 Q2 Octopus Deploy release includes a new feature allowing projects to be exported, and imported into another Space.** 

Octopus 2019.1 introduced [Spaces](https://octopus.com/docs/administration/spaces), a way to partition your Octopus Server.  What wasn't included was the ability to _move projects between Spaces_.  

The reason we didn't originally provide this functionality is because it proved really tricky to design and build.  The demand for this has been strong, with the message being that the value of Spaces is reduced if there is no way to partition existing projects into new Spaces. 

Our Customer Solutions team stepped up to fill the gap with the [Space Cloner](https://github.com/OctopusDeployLabs/SpaceCloner) project, a collection of PowerShell scripts to help with copying between Spaces.  This was useful in many cases, but came with limitations that can't be avoided by a solution that operated via the HTTP API as opposed to being "in the box". 

Fast-forward two years, and there is another reason many people want this ability: to migrate projects from self-hosted Octopus instances to [Octopus Cloud](https://octopus.com/docs/octopus-cloud).  

The 2021 Q2 release introduces the ability to export and import projects as a fully-supported, in-the-box feature! 

![Project Export/Import menu](import-export-menu.png "width=500")

One or more projects can be selected to be exported. A password is supplied to be used for encryption, as the export will include sensitive variables. 

![Export projects page](export-projects-page.png "width=500")

The export runs as a task, and results in a zip file which can be downloaded.  

The zip file can then be imported into another Space on either the same Octopus Server, a different server, or even an Octopus Cloud instance.  

![Import projects page](import-projects-page.png "width=500")

Similar to the export, the import runs as a task (it can take some time for large exports).

The selected projects are of course included, along with everything they require, including environments, tenants, variable sets, step templates, accounts, and certificates.  

There are notable things which are _not_ included: 
- [Deployment Targets](https://octopus.com/docs/projects/export-import#deployment-targets). These will need to be reconfigured in the target Space. The reason for this is there are likely to be networking considerations here, and Tentacles need to be explicitly configured to trust another Octopus Server.  
- [Packages](https://octopus.com/docs/projects/export-import#packages). It is not uncommon for self-hosted projects to have tens or even hundreds of gigabytes of packages. Including these in the export zip would not be practical. In many cases, old packages may not be required, and the build server can simply be pointed at the new Octopus Server. For cases where packages are required, we have provided an [example script](https://github.com/OctopusDeploy/OctopusDeploy-Api/blob/master/REST/PowerShell/Feeds/SyncPackages.ps1) which demonstrates syncing packages via the Octopus API.  

This is the first iteration of the Project Export/Import feature, and was designed primarily for one-time import/exports of a project.  This is useful for the following scenarios:
- Moving a project from a self-hosted instance to a cloud-hosted instance
- Moving a project between two spaces on the same instance, for example when splitting a space into multiple spaces 

This iteration _does not_ address repeatedly moving a project between spaces, for example to test upgrades or promote a project to a secure, segregated Octopus instance.  These scenarios are different in subtle but significant ways, often requiring different variable values, lifecycles, tenants, etc, to be maintained between the instances.  We have laid the foundations with this feature to support this scenarios, and hope to do so in a future release. 

The Project Export/Import feature is available in Octopus Cloud instances now, and will be in the 2021.1 release available from https://octopus.com/downloads 

We hope this will make migrating projects from self-hosted to Octopus Cloud much easier :) 