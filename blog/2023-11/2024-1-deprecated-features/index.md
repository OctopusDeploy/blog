---
title: Azure Cloud Services Deprecation
description: Azure will shortly be shutting down support for running Azure Cloud Services (Classic) which means it will also no longer be a viable Octopus Target
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
Although the latest version of [Tentacle](https://octopus.com/docs/infrastructure/deployment-targets/tentacle/windows/requirements#windows-server) has a minumum Windows requirement of Server 2012, we suprisingly still support workloads to legacy Tentacle agents running on Windows Server 2003. In order to modernise our execution engine we are starting to realign our supported platforms to those supported by the vendor themselves. 

Our first step in that direction is the dropping of the support for running workloads on Windows Server 2003 Targets and Workers.

For Windows Machines running Windows Server 2012 and below, this change upgrades the minimum requirement for .NET 462 to be installed. Note that this is not a requirement for later Windows versions or Linux targets as modern platforms instead utilize .NET Core.

Read the [blog post here](https://octopus.com/blog/deprecating-win2003) for more background on this change.

## Mono

Read the [blog post here](https://octopus.com/blog/deprecating-mono) for more background on this change.

## Azure Cloud Services
https://octopus.com/blog/azure-management-certs

One of the first cloud products provided that Octopus Deploy provided first class support for, was with [Azure's Cloud Services in 2018](https://octopus.com/blog/octopus-azure-deployments). 

Since that time, Azure have introduced dozens of new products and capabilities, many of which Octopus followed up with relevant built-in capabilities. In the meantime however, Azure have announced the sunsetting of the original Cloud Service resource, renamed Cloud Services Classic, with the [final retirement date set as August 31, 2024](https://learn.microsoft.com/en-us/lifecycle/products/azure-cloud-services-classic). In a little over 6 months teams, that are still relying on this cloud service will be unable to deploy to it at all, with Octopus Deploy or otherwise.

In preparation for this, over the next six months we will begin to gradually deprecate our support for Azure Cloud Servie Target, Azure Cloud Service Step or Management Certificate account type in new versions of Octopus Server. These Management Certificates have since bee repleased by Azure by the Resource Service authentication model.

### Migration
There are several migration options available depending on your intended usage. The reccomended approach is to migrate to using Azure Cloud Services Extended Support. While this will allow you to continue using your existing application code with minimal changes, the changes require to the deployment model and its apparent temporary nature means that Octopus is unlikely to support it with the same first-class level that it did for the Clasic services. 

Depending on your usages you may find that converting to Azure Service Fabric or Azure Web Services are viable approaches, and these currently enjouy first class support in Octopus Deploy through custom targets and steps.

### Timeline
1. From Octopus Server version 2024.1, we will begin logging and displaying warning whenever Azure Cloud Servie Target, Azure Cloud Service Step or Management Certificate account types are used. The intention is to give customers still relying on this technology a gentle reminder that they will need to invest in an alternative solution before Azure turns off support.

2. From Octopus Server version 2024.2 onwards we

## F#
