---
title: "What are tenanted deployments?"
description: Learn about the benefits and problems of tenanted deployments, plus how Octopus solves those problems.
author: andrew.corrigan@octopus.com
visibility: public
published: 2023-08-14-1400
metaImage: blogimage-whataretenanteddeployments-2023.png
bannerImage: blogimage-whataretenanteddeployments-2023.png
bannerImageAlt: Octopus infinity light cycles speeding out of Octopus Deploy to deliver to thousands of tenants.
isFeatured: false
tags: 
  - DevOps
  - Multi-Tenancy
---

As we explained in our post, [what is multi-tenancy?](https://octopus.com/blog/what-is-multi-tenancy), multi-tenancy is where software or its infrastructure splits into manageable chunks called 'tenants'. These splits can save costs, simplify processes, or allow customization per customer or destination.

Delivering multi-tenancy software can get very complex very quickly. It's common to manage that complexity with a tenanted deployment strategy.

Tenanted deployments see you use similar but separate deployment processes to manage tenant differences. This strategy offers benefits for multi-tenancy software but can cause problems in other areas.

In this post, I explore the benefits and problems of tenanted deployments, and explain how Octopus solves those problems.

## Benefits of tenanted deployments

### Tenant customization is easier

When developing a multi-tenancy application, your tenants will have differences no matter how you define them. For example, your customers might have different compliance needs, branding, or features.

As tenanted deployments mean separate delivery processes for each tenant, deployments become easily customizable.

If you sell book-tracking software to libraries, for example, tenanted deployments can help you ensure each library has its own logo and features.

### Deploy the same app to any infrastructure type or architecture 

Tenanted deployments let you deploy the same application anywhere as you can tailor the process for each target type.

You can send your software to a customer's on-premises data center *and* to another's preferred cloud platform. The flexibility of tenanted deployments makes that possible.

## Problems with tenanted deployments

### Process duplication

When doing manual tenanted deployments (or even using another deployment tool), you need to duplicate and customize a templated or existing process.

That might be fine if you only need to set up your tenants once. Add new tenants or make changes often, though, and your templates may drift and change drastically over time. That's almost certain to cause you problems later on.

### Risk of error iteration

If you do accidentally create a problem by changing a deployment process for new tenants, it may not be instantly obvious. You then risk error iteration as processes get copied repeatedly for future tenants.

As most developers know, it's hard to pinpoint the cause of a problem if you unknowingly introduced it months or years before.

### Harder to scale

Tenanted deployments are easy to manage if you're only dealing with a few tenants, but what if you need to scale suddenly?

The simple task of copying and adapting a process for a new tenant becomes a huge time sink when you need to do it for hundreds or thousands of tenants.

That's hours of copying and adapting scripts or duplicating deployment processes. That alone adds to the potential for mistakes.

### Releases become harder to track

When delivering software with tenanted deployments, tracking your releases can be challenging. It's even worse if deploying manually or with scripts alone.

Need to understand the state of play for every tenant? You'll likely need to trawl through mountains of infrastructure logs to understand what deployed where.

Now imagine you're dealing with hundreds or thousands of tenants. That's a lot of data to sift through to understand the overall status of your application.

## Octopus gives you the benefits of tenanted deployments without any of the problems

Octopus simplifies and automates any deployment process, and tenanted deployments are no different. In fact, our built-in Tenants feature practically removes all downsides of tenanted deployments.

Octopus's Tenants feature lets you:

- Use one deployment process for all tenants
- Add new tenants quickly and customize the deployment process using variables
- Automate infrastructure provisioning and teardown and perform routine maintenance tasks
- See exactly where you're at with a glance at the Tenants dashboard

## What next?

Check out our [Tenants use-case page](https://octopus.com/use-case/tenanted-deployments) to see how other organizations simplified their tenanted deployments with Octopus.

Happy deployments!