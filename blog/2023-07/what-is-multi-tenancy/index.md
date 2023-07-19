---
title: What is multi-tenancy?
description: We expore the various definitions of the term 'multi-tenancy'.
author: andrew.corrigan@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - tag
---

'Multi-tenancy' was once a term that described a very particular software architecture: Where an application lives on a server for other computers - tenants - to use it.

Over the years, the definition expanded and now describes much more than that. Sometimes we use it for software architecture, and sometimes not. We also use it to explain how we serve or deploy the applications or even how users access them.

At its simplest, the software industry now uses 'multi-tenancy' to explain where software or its architecture splits into manageable chunks. These chunks could save money, simplify processes, or make things safer or easier for customers.

Rather than try to unravel the whole term, it's easier to explain it with the most common ways organizations define multi-tenancy.

## Split by customer, as with Software as a Service (SaaS) applications

A SaaS application is software delivered via a subscription model and usually accessed by a web browser. SaaS software providers manage the application's infrastructure so users don't have to.

Multi-tenancy in SaaS sees each customer get the same product but with separate resources from your other customers.

Your customers will have isolated:

- Space on your hosting platform
- Identity management and security
- Databases

Here, your tenants could be your customers or your infrastructure.

## Split by location or region, as with international enterprise organizations

This is where organizations, big enterprises, for example, may serve customers worldwide.

That usually means organizations must account for regional differences, whether that's region-specific:

- Content
- Languages
- Outrage windows
- Legal requirements

Here, your tenants could be your regional infrastructure.

## Split by business model, as with retail chains

Your business model could dictate where your application becomes multi-tenancy.

The best example is when delivering software to many brick-and-mortar locations. This is common for retail chains, hospitals, hotels, and more.

In this scenario, you could be: 

- Working for an international organization delivering software to worldwide branches
- An independent organization delivering software to physical locations for many organizations

Here, your tenants could be each store or a group of stores.

## Split by hosting platform, as with government or legal software

Due to strict processes or legal requirements, you may need to deliver your software to your customers' own hosting solutions.

For example, you may need to deliver your software to your customer's own:

- Cloud service
- On-premises servers or desktop computers
- Data Center
- Hybrid setups

Here, your tenants could be a target or group of targets you deploy to.

## And everything in between

At this stage, you may be thinking, '*our multi-tenancy application fits a couple of these descriptions*,' and you'd be absolutely right!

These scenarios can crossover, for sure. And some apps may use more than one multi-tenancy variation at different stages.

That's because software development is only growing more complex and with it, the term evolves too. The lines we used to draw between these scenarios are blurry and will only get blurrier still.

## What next?

See Octopus's Tenants feature page for more information on how Octopus can help you deploy your multi-tenancy applications faster and more reliably.