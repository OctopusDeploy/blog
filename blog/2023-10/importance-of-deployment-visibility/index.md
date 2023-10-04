---
title: Why visibility is important for modern deployments
description: We look at the reasons why reporting and visibility are important for modern deployments 
author: andrew.corrigan@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - DevOps
  - Multi-Tenancy
  - Rollbacks
---

Nowadays, DevOps teams are spoilt for choice for tooling. There are countless options to solve every problem, each with different focuses and methodologies.

When talking about solvable problems, however, there's one challenge often undersold. That's the reporting and visibility of deployments.

In some cases, it's easy to understand how good reporting became so underappreciated. Historically, software had only periodic updates to a single server or target, meaning it was easier to know where your software was at. These kinds of deployments are pretty rare in 2023, though. Deployment pipelines now have so many moving parts in the form of environments, architectures, and other components.

Using a [tenanted deployment strategy](https://octopus.com/use-case/tenanted-deployments) to serve multi-tenant or multi-customer software, for example? Well, understanding your deployments can feel like you're untangling that big bunch of cables you keep 'just in case.' (though not if you use Octopus - we'll get to that at the end.)

But why is deployment visibility more important now than ever? Let's explore!

## There are more reasons for reporting now

The reason for even reporting on deployments in the first place has changed a lot over the years. As little as a decade ago, software teams only really cared about deployment success or failure so they could fix a problem or roll back to previous versions.

Nowadays, there are many more reasons to report on deployments. For example, you can use deployment visibility to:

- Inform your software strategies
- Track the releases your tenants or customers have
- Monitor infrastructure
- Find good or bad patterns
- Improve processes
- Allow other interested parties to see project progress

## Non-developers need to understand deployment progress too

When deployment feedback was only for technical people who cared about success or failure, they collected data in log files. Log files are still important in understanding problems, but they're not easy to read to everyone who needs deployment visibility.

Managers or team leaders, for example, typically won't need every fine detail. Yet, they probably need to understand the deployment status at a high level to manage projects, track progress, or shift resources as needed.

Other parties may need more information than managers, but not need *all* the details. For example, operations teams would likely want infrastructure-related deployment info so they can keep everything healthy and ticking over. Software testers may need to track releases across environments to know what they're testing and why.

Visibility is even useful for frontline support, too. It can allow those dealing directly with customers to spot and supply relevant information that saves others' time.

## If you don't track your deployments, you won't know how to improve them

One of the core concepts of DevOps and related methodologies, like Continuous Integration and [Continuous Delivery (CI/CD)](https://octopus.com/devops/continuous-delivery/), is you should always try to improve your deployment pipeline.

If you don't have a solid understanding of your deployment process, you can't possibly know where to improve. Good visibility over your deployments can help you:

- Find new things to automate
- Remove unimportant or unnecessary steps
- React to failures much quicker
- See what tooling isn't working for you

With good, easy-to-understand data, finding those areas of improvement becomes much easier.

## Octopus is *really* good at deployment visibility, fyi

Many deployment tools (or CI platforms used as deployment tools) treat reporting as an afterthought. Octopus, however, understands that good deployment reporting saves time for everyone. And for software providers, time saved is money saved.

To help you check progress, spot problems, and find areas for improvement, Octopus delivers key information in a few ways:

- **Octopus dashboard** - The dashboard is the first thing you see when you log in. You can see your entire deployment strategy at a glance and drill down for as much detail as you need. [See our sample dashboard](https://samples.octopus.app/app#/Spaces-682).
- **Tenants dashboard** - Useful for those using a tenanted deployment strategy. This focused dashboard sorts your deployments by tenant or customer to track deployments to all tenants, no matter how many you have. [See our sample Tenants dashboard](https://samples.octopus.app/app#/Spaces-682/tenants).
- **Insights dashboard** - If you're an enterprise customer, you can use Octopus's Insights feature to see how your deployments perform against 4 key [DORA metrics](https://octopus.com/devops/metrics/dora-metrics/). This can help you identify areas for improvement in your DevOps adoption. [See our sample Insights dashboard](https://samples.octopus.app/app#/Spaces-682/insights/reports/InsightsReports-1/overview).

Happy deployments.