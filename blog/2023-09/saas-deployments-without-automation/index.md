---
title: How SaaS tenanted deployments would look without automation
description: We explore how difficult SaaS tenanted deployments would be if we didn't have automation.
author: andrew.corrigan@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: img-mtcampaign-saastenanteddeploymentwithoutautomation-2023.png
bannerImage: img-mtcampaign-saastenanteddeploymentwithoutautomation-2023.png
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - DevOps
  - Multi-Tenancy
---

Continuous Integration and Continuous Delivery's recommendation of deployment automation is *hugely* important for large organizations with complex software. That's especially true for multi-tenancy software delivered with [tenanted deployments](https://octopus.com/blog/what-are-tenanted-deployments).

But what if we didn't have deployment automation? How would tenanted deployments even work?

In this, the second of a trio of posts, we ask what tenanted deployments for Software as a Service (SaaS) applications look like with manual processes.

## A common SaaS tenanted deployment scenario

You work for a company that makes a SaaS timekeeping application for schools that helps them arrange assemblies and other school events. School administrators access the software through a web browser.

Schools can also:

- Tailor the application from a selection of additional features
- Customize your software's look and feel their school's branding

## How the deployments would work without automation

SaaS providers would have the following things to consider without deployment automation.

### Infrastructure becomes a bigger consideration

As a SaaS provider, you worry about how to host and update your software so your customers don't need to.

To host the software, you have a couple of options. The most likely choice in 2023 is hosting your app with a cloud provider. You would give each customer either:

- A piece of your infrastructure shared with - but isolated from - your other customers
- Their own cloud infrastructure under your ownership and management

Each customer in this scenario is a 'tenant.'

There are countless methods and cloud options to deliver software, from virtual machines to containers. Your choice must best balance the needs of your application, customers, and company.

Another alternative to the cloud includes hosting your application on physical hardware on your premises or at a server farm. Due to hardware costs, the space needed, and the difficulty of scale, that's unlikely these days.

Unlikelier still is keeping hardware at your customer's locations, which wasn't unheard of before SaaS as we know it now. In this scenario, you'd need to install a server or computer in every school that signs up for your software.

Given physical hardware options are rare for a modern SaaS model, we won't spend much time on it here. Suffice to say, if you don't have automation, physical hosting causes deployment problems most SaaS providers don't even think about. If you use the cloud, you'll never have the same travel headaches we mention in the companion posts about other scenarios, for example.

But while infrastructure setup isn't strictly part of the deployment, it's important to talk about for SaaS deployments. With the cloud, setting up new customers and deploying your software universally involves creating infrastructure that wouldn't exist otherwise.

Plus, if you ship your software with containers, to deploy updates *is* to tear down the infrastructure hosting your app and replace it with new infrastructure. That's to say, SaaS providers can largely consider infrastructure part of their deployment process.

Manually and repeatedly setting up cloud infrastructure is a *pain*. Major providers don't only offer multiple hosting options but also multiple versions of those options. When setting up a new customer, you navigate countless menus and select from jargon-described boxes. It's a time-consuming task made worse when your business scales.

There's a lot of technical risk, too. You might make a setup mistake that doesn't affect your software right away but might limit you later. A concentration lapse could cause performance problems your other customers might not experience. Choosing a similar-but-wrong option might also see you pay more on infrastructure for one customer than another.

A good deployment tool, however, can take care of infrastructure setup for you and help standardize your environments.

### Scale becomes both blessing and curse

The biggest problem for SaaS applications is also the thing you want the most: scale.

Strike gold and build a popular app, and you could quickly find yourself inundated with new customers. In the early days, your software could serve just 5 to 10 schools, but at any stage, you could get swamped if word gets around.

What if your product attracts the attention of state or national Education Departments? That could be your golden ticket, but it also means thousands of new tenants to create and support.

SaaS software can go from tens to thousands of customers very fast. That's absolutely what you want, but your methods for growth need to be sustainable, and the deployment process is where things can fail. Your team processes, infrastructure, and deployment strategies *must* be able to scale.

With deployment automation, scaling is simpler and costs easier to control against your profit margin. Without it? Not only does time spent creating infrastructure balloon, but updates for existing customers could also become impossible.

## The result

Though the challenges differ slightly to what we discussed in previous post, the results are largely the same. Without deployment automation, whatever update strategy you pick will result in:

- Much slower software delivery - bad for bug or vulnerability fixing and delivering new features
- A higher risk of technical problems - bad for customer relations

And in this scenario, where your customer base could expand rapidly, you can't really afford either.

Thankfully, deployment automation *does* exist. But not all deployment tools handle tenanted deployments as well as Octopus does. Octopus can also help you with cloud infrastructure setup. Octopus's Runbooks feature can help you spin up, tear down, and manage infrastructure alongside or as part of your deployments.

Why not check out our [tenanted deployments use-case page](https://octopus.com/use-case/tenanted-deployments) to find out why.