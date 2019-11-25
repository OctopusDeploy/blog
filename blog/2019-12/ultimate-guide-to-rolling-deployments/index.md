---
title: The Ultimate Guide to rolling deployments
description: What are rolling deployments and why they are useful? This post covers rolling deployment patterns and how to use them.
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
If the majority of your customers are asleep, then that's probably acceptable. But what happens if your customers are using your applications 24-7? Today, it's increasingly common to expect systems to always be online and there are a few different deployment strategies you can use to achieve this. In this article I'll discuss one of these strategies in more depth; Rolling deployments.

<h2>In this post</h2>

!toc

## What are rolling deployments?

A rolling deployment is a deployment pattern (also known as an incremental, batched or ramped deployment) where new software is delivered, usually to a small subset of deployment targets at a time, until all of the targets have the updated version of software rolled out.

**[IMAGE?]**

This incremental approach is often favoured in web applications which sit behind a load balancer, as this allows the application instances not being updated, to remain online serving traffic.
The rolling deployment typically consists of the following steps:

For-each instance:

 1. Take out of load-balancer pool
 2. Stop application
 3. Deploy new version of application
 4. Verify deployment successful
 5. Start application
 6. Return into load-balancer pool

Although the scenario above describes a web application rolling deployment, it's equally possible to achieve rolling deployments for other types of application, providing they are built in a way which supports ending their process safely.

For example, In an Octopus Deploy [High Availability](https://octopus.com/docs/administration/high-availability) configuration, there is a [drain](https://octopus.com/docs/administration/high-availability/managing-high-availability-nodes#ManagingHighAvailabilityNodes-Drain) option, which prevents any new tasks from executing, and finishes up any tasks it's currently executing until idle. Features like draining allow for the safe termination of a process, which can then be updated and brought back online.

## Why are they useful?

Rolling deployments have a number of benefits over other patterns (canary, blue/green)

### Incremental update
 
New versions are rolled-out incrementally. This allows you to verify that the application is working as more traffic is directed to your newly deployed software.

The staggered approach to roll-outs mean that in the unlikely event that you need to initiate a rollback, you can do that in a controlled, incremental way.

### Keeping the lights on

Whilst you go about updating a small number of your application instances, the remaining instances are online, available to 

### Parallelism

Depending on what's performing the rolling deployment, there are often parameters to allow you to control the number of concurrent instances that are updated at one time.

## Rolling deployment patterns [HOWTO]

TODO

## Kubernetes Rolling updates

TODO

## A word on the database

The proverbial elephant in the room I haven't discussed yet, is the database for your application. Performing rolling deployments with a central shared database can be tricky, but not impossible. 
If you wanted to perform rolling deployments with database changes involved, then I'd recommend deploying the database first. You would need also need to make it backwards compatible with any previous versions of code you have deployed.

We have a series of posts on [database deployments](http://octopus.com/database-deployments) that go into more detail.

## Wrapping up

As with most things, rolling deployments is just one pattern available in your toolset to optimise deployment of your software. But with an incremental approach, it allows you to keep your  Feel free to leave us a comment, and let us know what you think about rolling deployments!
