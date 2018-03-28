---
title: "Azure Cloud Deployments with Octopus"
description: "Kicking off our Azure Cloud Deployments with Octopus blog series looking at the breadth of options currently available."
author: rob.pearson@octopus.com
visibility: private
tags:
 - Azure
---

Octopus started with automating deployments in the Microsoft ecosystem and we long supported deployments to Microsoft Azure platform as it evolved. Buckle up as we take a trip down memory lane.

## Early Days

It all started with Octopus Deploy v1.5.0.1645 (launched in March 2013 - 5 years ago!) which first added support [deploying to Windows Azure specifically Cloud Services](https://octopus.com/blog/octopus-1.5-azure-ftp-scriptcs). This was fantastic and and helped tons of teams start deploying to the cloud and move to repeatable and reliable deployments. Microsoft continued to introduce new services include Azure web apps Platform as a Service (PaaS). This proved very popular and one of my first Octopus automated deployments was deploying an ASP.NET Web API web service to an Azure web app using Octopus 2.6 and [this blog post](https://octopus.com/blog/deploy-aspnet-applications-to-azure-websites). 

## First Class Support and a Pivot

In June 2015, Octopus 3.0 launched and introduced Azure accounts and new deployment targats to deploy to Azure Cloud Services and Azure Web web apps and as well as a new step to execute Azure Powershell scripts. We pivoted shortly after to move to from deployment targets to first-class steps to better support binding and changing values through environments. We contined to add more and more first-class step types including support to deploy [Azure Resource Group Templates](https://octopus.com/blog/octopus-deploy-3.3), [Service Fabric](https://octopus.com/blog/octopus-release-3-13) and most recently [Azure functions](https://octopus.com/blog/azure-functions).

## Octopus + Azure vNext

We're gearing up to ship the next generation of Octopus support for Azure deployments and infrastructure automation. Very exciting features and new steps are coming soon!  There's some great posts coming so be sure to check out our blog each week.

Blog series posts:

* TBA 
* TBA
* TBA
* TBA