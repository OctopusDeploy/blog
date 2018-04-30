---
title: Octopus April Release 2018.5
description: Octopus 2018.5 features our next generation of Azure support.
author: rob.pearson@octopus.com
visibility: private
metaImage: metaimage-shipping-2018-5.png
bannerImage: blogimage-shipping-2018-5.png
published: 2018-05-01
tags:
 - New Releases
---

![Octopus Deploy 2018.5 release banner](blogimage-shipping-2018-5.png)

This month, our headline feature is Azure. We have new deployment targets for Azure Web Apps, Cloud Services and Service Fabric Clusters, plus the ability to manage them in Octopus using PowerShell Cmdlets.

## In This Post

!toc

## Release Tour

<iframe width="560" height="315" src="https://www.youtube.com/embed/TODO" frameborder="0" allowfullscreen></iframe>

## PaaS Targets for Azure

![New Deployment Targets](new-targets.png "width=500")

This release reintroduces both **Azure Web Apps** and **Azure Cloud Service** modeled as deployment targets, and introduces **Azure Service Fabric Clusters** to the deployment target family.

For more information on the PaaS Targets, have a read of our previous [blog post](https://octopus.com/blog/paas-targets). 

## Managing Octopus Infrastructure

One of the problems with our initial implementation of **Azure Web App** Targets was that there was no easy path to create them as needed. You could easily create the Azure Web App, but Octopus required manual steps to represent them in Octopus.

Now you can create those new **PaaS targets** as easily as Azure resources, using our new built-in PowerShell Cmdlets.

The initial release of these cmdlets will allow you to create **Azure Accounts** for all the new Azure targets, and be able to delete those targets too.

For more information, see the [PaaS blog post](https://octopus.com/blog/paas-targets) and the [documentation](https://octopus.com/docs/infrastructure/dynamic-infrastructure). Also, look for more blog posts and [Will it Deploy](https://www.youtube.com/watch?v=tQb8PJ0jzvk&list=PLAGskdGvlaw13QRF-ypT9h83QTPutlbMn) videos.

## Azure Accounts as Variables

You can now set your Azure Accounts as Variables, for both project variables, tenant variables and library variable sets.

This will give you the benefit of being able to scope different accounts across each environment or tenant. You can still select an account directly on the Azure PowerShell step, just as you have always done, but now you can also bind this to an Azure Account Variable.

The properties of the Azure Account (Client Id, Subscription Id, Tenent Id etc.) will also be available to use in your scripts. Just turn on the `OctopusPrintVariables` option to [see all the variables available](https://octopus.com/docs/support/debug-problems-with-octopus-variables#DebugproblemswithOctopusvariables-Writethevariablestothedeploymentlog).

## Breaking Changes

We have upgraded the Azure SDK library and the Azure PowerShell modules to support the latest Azure features. Most notably missing was support for nested ARM templates, which will now work out of the box.

These upgrades have also forced the minimum supported environment for Octopus to **.Net 4.5.2** and **PowerShell 5.0**.

## Upgrading

As usual [steps for upgrading Octopus Deploy](https://octopus.com/docs/administration/upgrading) apply. Please see the [release notes](https://octopus.com/downloads/compare?to=2018.5.0) for further information.

## Wrap up

That’s it for this month. Feel free leave us a comment and let us know what you think! Go forth and deploy!
