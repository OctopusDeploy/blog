---
title: Octopus July Release 2018.7
description: Octopus 2018.7 - Sharing the Workload with Workers!
author: rob.pearson@octopus.com
visibility: public
metaImage: metaimage-shipping-2018-7.png
bannerImage: blogimage-shipping-2018-7.png
bannerImageAlt: Octopus Deploy 2018.7 release banner
published: 2018-07-18
tags:
 - Product
---

![Octopus Deploy 2018.7 release banner](blogimage-shipping-2018-7.png)

This month, our headline feature is **Octopus Workers** which allows you to shift deployment work off your Octopus Server.  This change brings several benefits including improved security, better performance, and other cool things like the ability to create pools of machines dedicated to specific tasks.

## In This Post

!toc

## Release Tour

<iframe width="560" height="315" src="https://www.youtube.com/embed/N4uBvgB3ehM" frameborder="0" allowfullscreen></iframe>

## Octopus Workers and Worker Pools

Octopus workers and worker pools allow you to shift deployment work off your Octopus Server. Octopus has always had a built-in worker but we never called it that. Any time you did deployment work that executed a script on the Octopus Server, it used the integrated worker. This work includes Azure and AWS steps as well as scripts steps where you specified to run on the Octopus Server itself.

With the introduction of workers and worker pools, you now have the option to move this work to a separate machine as well as create pools of workers to utilize for special purposes. Nothing has changed about how steps are executed; workers provide an option about **where** those steps are executed.

Octopus workers bring many benefits including the following:

* Improved security as custom scripts aren't running on your Octopus Server.
* Better performance for your Octopus instance as well as project deployments.
* Other cool stuff like creating worker pools to handle specific tasks like cloud deployments, database deployments and any other particular tasks where you may have dependencies that would benefit from a dedicated set of machines.

Read more about this in our [Octopus Workers blog series](https://octopus.com/blog/tag/Workers).

### Licensing

> Are workers and worker pools limited by license? They are kind of like deployment targets, right?

At this point we've decided to limit the use of workers and worker pools for different license kinds:

1. Customers using a grandfathered `Community` (free), `Professional`, `Team`, or `Enterprise` license are limited to one worker. This enables you to transition from using the built-in worker into [a more secure configuration](https://octopus.com/docs/administration/security/hardening-octopus#configuring-workers).
1. Customers using a grandfathered `High Availability` license get **unlimited** workers.
1. Customers with one of the current `Standard` or `Data Center` licenses also get **unlimited** workers. _These licenses were introduced around the Octopus 4.0 timeframe._
1. [Octopus Cloud](https://octopus.com/cloud) customers using a `Standard` subscription get **unlimited** workers.
1. [Octopus Cloud](https://octopus.com/cloud) customers using a `Starter` subscription are limited to one worker. It's just for getting started after all.

We may decide to change these limits over time. If you have any questions about your license and how it applies to workers, please contact [sales@octopus.com](mailto:sales@octopus.com).

## Perf and Polish Improvements

We’re also shipping some performance and polish improvements. That is, we’ve made some significant updates to improve Octopus Server performance and usability. Particularly for larger installations, including much lower CPU usage on SQL server in some cases, improvements to deletion, and faster project and infrastructure dashboards. We’re continually working to improve our user experience, and this month, we’ve tweaked the variable snapshot update process as well as improving lifecycle, channel, and role scoping pages.

## Azure Web Sites and Slots

This release, we have reinstated first class support for **Azure Web Site Deployment Slots**, allowing you to select a specific slot for a **Azure Web App** deployment target or setting the slot directly on a **Deploy an Azure Web App** step. We have also improved the performance of the **Azure Web Site** selector.

## Breaking Changes

The API endpoint for listing Azure Web Sites `\api\accounts\{id}\websites`, no longer returns the Deployment Slots, only the production Web Sites. There is a new API endpoint `/api/accounts/{id}/{resourceGroupName}/websites/{webSiteName}/slots` for listing the Deployments Slots for a single Web Site.

## Known issues

When using [Dynamic Infrastructure PowerShell cmdlets](https://octopus.com/docs/infrastructure/dynamic-infrastructure/azure-web-app-target) to create a new Azure PaaS deployment target, if a subsequent step deploys a package to the newly created target, from an external worker, the deployment will fail. Adding a **Health Check** step, configured for a **full health check**, between the script step and the package deployment step which will allow the deployment plan to acquire the necessary packages to the worker. We've created a [GitHub issue](https://github.com/OctopusDeploy/Issues/issues/4731) to track this problem until it's resolved.

## Upgrading

As usual [steps for upgrading Octopus Deploy](https://octopus.com/docs/administration/upgrading) apply. Please see the [release notes](https://octopus.com/downloads/compare?to=2018.7.0) for further information.

## Wrap Up

That’s it for this month. Feel free to leave us a comment and let us know what you think! Go forth and deploy!
