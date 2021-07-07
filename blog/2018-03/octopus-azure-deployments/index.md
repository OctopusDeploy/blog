---
title: "Azure Cloud Deployments with Octopus"
description: "Kicking off our Azure Cloud Deployments with Octopus blog series looking at the breadth of options currently available."
author: rob.pearson@octopus.com
visibility: public
published: 2018-03-30
metaImage: metaimage-azure.png
bannerImage: blogimage-azure.png
bannerImageAlt: Octopus Juggling Azure Balls
tags:
 - Engineering
---

![Octopus Juggling Azure Balls](blogimage-azure.png)

Octopus Deploy started with automating deployments in the Microsoft ecosystem and we have long supported deployments to the Microsoft Azure platform as it evolved. Buckle up as we take a trip down memory lane.

## Early Days

It all started with Octopus Deploy `v1.5.0.1645` in March 2013 (5 years ago!!), which first added support for [deploying to Windows Azure specifically Cloud Services](https://octopus.com/blog/octopus-1.5-azure-ftp-scriptcs). This was fantastic and helped tons of teams start deploying to the cloud and move to repeatable and reliable deployments. Microsoft continued to introduce new services including Azure web apps Platform as a Service (PaaS). This proved very popular and one of my first Octopus automated deployments was deploying an ASP.NET Web API web service to an Azure web app using Octopus 2.6 and [this blog post](https://octopus.com/blog/deploy-aspnet-applications-to-azure-websites).

## First Class Support and a Pivot

In June 2015, Octopus `3.0.0` launched and introduced Azure accounts and new deployment targets to deploy to Azure cloud services and Azure web apps, as well as a new step to execute Azure PowerShell scripts. We pivoted shortly after to move from deployment targets to first-class steps to better support binding and changing values through environments. We continued to add more and more first-class step types including support to deploy [Azure Resource Group Templates](https://octopus.com/blog/octopus-deploy-3.3), [Service Fabric](https://octopus.com/blog/octopus-release-3-13), and most recently [Azure functions](https://octopus.com/blog/azure-functions).

## Octopus + Azure vNext

We're gearing up to ship the next generation of Octopus support for Azure deployments and infrastructure automation. Very exciting features and new deployment targets are coming soon!  There's some great posts coming so be sure to check out our blog each week.

Blog series posts:

* [PaaS Deployment Targets](/blog/2018-04/paas-targets/index.md)
* [Service Fabric Deployment Targets](/blog/2018-05/service-fabric-cluster-targets/index.md)
* [Managing Dynamic Targets](/blog/2018-05/dynamic-infrastructure/index.md)
