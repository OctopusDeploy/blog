---
title: The Ultimate Guide to rolling deployments
description: What are rolling deployments and why they are useful? This post covers the rolling deployment pattern and how to use it.
author: mark.harrison@octopus.com
visibility: private
published: 2020-02-01
metaImage: 
bannerImage: 
tags:
 - DevOps
---

![Rolling Deployments](rolling-deployments.png)

Whilst you can deploy new versions of your application, such as a web site by bringing the whole site offline, the question is, what's the impact?
If the majority of your customers are asleep, then that's probably acceptable. But what happens if your customers are using your applications 24-7? Today, it's increasingly common to expect systems to always be online and there are a few deployment patterns you can use to achieve this. In this article I'll discuss one of these patterns in more depth; Rolling deployments.

!toc

## What are rolling deployments?

A rolling deployment is a deployment pattern (also known as an incremental, batched or ramped deployment) where new software is delivered, usually to a small subset of deployment targets at a time, until all of the targets have the updated version of software rolled out. 

A rolling deployment typically consists of the following process:

 Drain the first set of nodes and take them out of the load-balancer pool

![Rolling Deployment: Draining nodes](rolling-deploy-1.png)

 Stop application, deploy new version of application and (optionally) verify deployment successful

 ![Rolling Deployment: Update nodes with new versions](rolling-deploy-2.png)

Once the first nodes have updated successfully, proceed with draining any additional nodes, whilst your new version is now online serving traffic.

 ![Rolling Deployment: Drain remaining nodes in pool](rolling-deploy-3.png)

 Stop the application on the remaining nodes, deploy new version of application and (optionally) verify deployment successful again

 ![Rolling Deployment: Update remaining nodes in pool](rolling-deploy-4.png)
 
 Once the application has been deployed successfully, you now have version 2 of your software running successfully.

![Rolling Deployment: Update remaining nodes in pool](rolling-deploy-5.png)

This incremental approach is often favoured in web applications which sit behind a load balancer, as most load balancers support a concept known as `Connection draining`. This is simply allowing connections to a service to finish naturally, as well as preventing any new connections. 

By performing this draining action, this allows the instances that are selected to be updated to be removed from the available pool, whilst a number remain online serving traffic.

:::hint Although the scenario above describes a web application rolling deployment, it's equally possible to achieve rolling deployments for other types of application, providing they are built in a way which supports ending their process safely.
:::

For example, Octopus Deploy's [High Availability](https://octopus.com/docs/administration/high-availability) configuration also has a [drain](https://octopus.com/docs/administration/high-availability/managing-high-availability-nodes#ManagingHighAvailabilityNodes-Drain) option, which prevents any new tasks from executing, and finishes up any tasks it's currently executing until idle. Features like draining allow for the safe termination of a process, which can then be updated and brought back online. 

## Why are they useful?

So why use rolling deployments over other patterns (canary, blue/green)? Well, rolling deployments offer the following benefits

### Incremental update
 
New versions of your application are rolled-out incrementally. This allows you to verify that it's working as more traffic is directed to your newly deployed software.

The staggered approach to roll-outs also means that in the unlikely event that you need to initiate a rollback, you can do that in a controlled, incremental way.

### Keeping the lights on

Whilst you go about updating a small number of your application instances, the rest continue to serve requests. This means there is no downtime for your application, as it is available for your users throughout the deployment.

### Controlled Verification

_TODO_

### Parallelism

Depending on what's performing the rolling deployment, there are often parameters to allow you to control the number of concurrent instances that are updated at any one time. Further deployments won't start until a previous deployment has finished.

:::hint
You can use the `Window size` option within an Octopus rolling deployment to control how many deployment targets can be deployed to at once.
:::

## Rolling deployment patterns [HOWTO]

The following sections show how to perform rolling deployments 

### Kubernetes Rolling updates

_TODO_

### Azure DevOps

_TODO_

### Octopus Rolling deploy

_TODO_

## A word on the database

The proverbial elephant in the room I haven't discussed yet, is the database for your application. Performing rolling deployments with a central shared database can be tricky, but not impossible. 
If you wanted to perform rolling deployments with database changes involved, then I'd recommend deploying the database first. You would need also need to make any changes to your database backwards compatible with previous versions of code you have deployed.

We have a series of posts on [database deployments](http://octopus.com/database-deployments) that go into more detail.

## Wrapping up

No matter which tool you are using, rolling deployments is just one pattern available in your toolset to optimise deployment of your software. But with an incremental approach, it allows you to keep your applications online whilst slowly rolling out newer versions of your software, making it a favourite of mine for minimal disruption.

Feel free to leave us a comment, and let us know what you think about rolling deployments!
