---
title: Cloud-nomad architecture
description: As more organizations rethink their cloud and microservices decisions, it's time for cloud-nomad architectures.
author: steve.fenton@octopus.com
visibility: public
published: 2023-06-21-1400
metaImage: 
bannerImage: 
bannerImageAlt: 
isFeatured: false
tags: 
  - DevOps
  - Cloud Orchestration
  - Microservices
---

A quiet revolution has occurred in the software industry, with many organizations backing away from cloud computing and microservices. These decisions are mainly influenced by cost control and performance.

In this post, we look at:

- The repatriation and consolidation trends
- Why it's a step forward, not a step back
- The core idea of cloud-nomad architecture

## The discovery process

When a new technology or technique arrives in the software industry, you must imagine its impact on the software you create. You can discover the benefits, limits, and costs only after using the new technology in many different scenarios. We shouldn't call this a  *hype cycle*. It's simply part of the discovery process.

Cloud computing and microservices have shared their discovery timeline. Both ideas were around for some time before they caught on, and they each grew rapidly between 2010 and 2020. The result of all that growth is that the industry has developed a stronger sense of where they work best and, most importantly, where they don't work well.

This has led to an increasing number of stories where organizations:

- Move applications away from the cloud to on-premises infrastructure, known as *repatriation*
- Consolidate microservices into fewer macroservices

Crucially, this trend doesn't mean we were wrong to use cloud and microservices. In some cases, we just went too far. Now it's time to rebalance.

### The repatriation trend

A survey by [Virtana](https://www.virtana.com/wp-content/uploads/2021/02/Virtana-StateofHybridCloud-Survey-Report_Feb2021_FINAL.pdf) found that 95% of organizations had started a cloud migration, but 72% went on to repatriate applications.

One of the key drivers is cost. Not only can on-premises infrastructure work out significantly cheaper over 5 years, but the costs are also more predictable. The cloud offers a seemingly infinite ability to scale. However, this can lead to unpredictable spending, especially with volatile costs such as egress charges.

Though cost is the top reason for repatriation, organizations often report improved performance running their applications on their on-premises infrastructure.

A crucial consideration for repatriation is that it's now possible to use cloud-native technology on your private infrastructure, like Infrastructure as Code (IaC) and containers. Moving applications back to on-premises infrastructure or private clouds is a further step forward rather than a return to old ways.

Dropbox was an early example of this trend, [repatriating over 600 petabytes of data](http://web.archive.org/web/20170629062600/https://insights.hpe.com/articles/cloud-or-on-premises-for-dropbox-the-answer-is-yes-1702.html) in 2016 for their US-based customers. At the time, they decided to continue using the cloud for European customers to satisfy data residency requirements.

More recently, 37signals, the creators of Basecamp and Hey, [repatriated workloads](https://world.hey.com/dhh/five-values-guiding-our-cloud-exit-638add47) with an estimated cost saving of $7 million over 5 years. They also found significant performance benefits as a result of this move.

You don't need to pick a single hosting strategy for all workloads. You can use a mixture of public cloud, private cloud, and on-premises infrastructure, just as you can use multiple cloud vendors.

### The microservice consolidation trend

With microservices, many companies are opting to consolidate them either into fewer *macroservices* or into a single monolithic application.

When organizations return to a monolith, they are taking care to ensure a loosely coupled architecture. When organizations move to macroservices, they pay attention to Conway's Law by organizing teams and services around business capabilities. A sprinkling of domain-driven design and team topologies is crucial here.

:::hint
Mel Conway wrote a 1967 paper titled [How Do Committees Invent](http://www.melconway.com/Home/Committees_Paper.html), where he made a social observation that you can summarize as:

> Any organization that designs a system (defined broadly) 
> will produce a design whose structure is a copy of the 
> organization's communication structure.

Fred Brooks shared this idea in *The Mythical Man Month*, and named it *Conway's Law*. This book is in our [DevOps reading list](https://octopus.com/devops/reading-list/#the-mythical-man-month-book).
:::

Microservices aim to trade some performance in exchange for operational benefits. You can deploy and scale small services independently, but they must pass data out-of-process to other services. There's some sophistication needed to monitor, debug, and troubleshoot a microservice architecture.

Organizations must know the trade-offs and take action when the benefits evaporate. For example, [Segment (Twilio)](https://segment.com/blog/goodbye-microservices/) found microservices were making it *harder* to change the code. Dependency management was a nightmare; they found it hard to manage the scaling.

Amazon Prime's [Video Quality Analysis team](https://www.primevideotech.com/video-streaming/scaling-up-the-prime-video-audio-video-monitoring-service-and-reducing-costs-by-90) consolidated the stream monitoring into a single service. This was a closer match to the team design. Within the service, the architecture is still loosely coupled, but they no longer need to pass the data over the wire to different services.

Many people will argue that macroservices represent "microservices done correctly". It might be more appropriate to say they are service-oriented architecture done right.

## What is cloud-nomad architecture?

With organizations drastically rethinking their approach to architecture and infrastructure, a cloud-nomad architecture provides a way to maintain maximum options.

A cloud-native architecture encourages application design that maximizes the benefits of cloud environments. Cloud-nomad architecture takes this further, requiring the software to be easily movable.

To be cloud-native, you use technologies like containers, microservices, and immutable infrastructure to build loosely-coupled systems that are easy to manage and operate. Cloud-nomad means you can shift the application between cloud vendors, private clouds, and traditional data centers.

Cloud-nomad architecture encompasses the trend toward fewer services you can host anywhere, thanks to modern cloud-native technology.

### Portable shelters

Ancient hunter-gatherers learned to move to where the food was. Rather than having a fixed home, they would migrate based on the seasonal availability of water, plants, and animals. In more modern times, tinkers and traders moved to where they could find new customers.

To maintain mobility, nomads developed portable dwellings or temporary shelters like [goahti](https://en.wikipedia.org/wiki/Goahti), [tipis](https://en.wikipedia.org/wiki/Tipi), and [wickiups](https://en.wikipedia.org/wiki/Wigwam).

The ability to move easily to a new location is central to cloud-nomad architecture.

### A step or two further than cloud-native

The CNCF definition of cloud-native encompasses public and private clouds. You should be able to run your cloud-native application on the public cloud, in a data center, or using on-premises infrastructure. Taking this a step further, to say that it should also be easy to move between these options, gives us cloud-nomad architecture.

To achieve this, you must avoid depending on vendor-specific features and embrace ephemeral infrastructure. You need infrastructure automation that works across different hosting scenarios.

Cloud-nomad architecture also encourages you to balance your microservice architecture against Conway's Law. The complexity will likely outweigh the benefits if you have 5 teams in your organization and 100 microservices. Many teams found that excessive numbers of services cost too much to run, caused performance problems, and made it harder to change their applications.

A cloud-nomad architecture has all the properties of cloud-native architecture. Additionally, it:

- Is minimally complex 
- Values portability by avoiding vendor-specific dependencies
- Automatically provisions infrastructure in a vendor-agnostic way

Like nomadic shelters, your application should be easy to pack up, move, and set up in a new location. When the shelter is too heavy, has too many parts, or depends on finding highly specific replacement parts, it becomes less portable.

## Conclusion

When designing software, it's common for developers to create an abstraction to make it easier to change the selected database. In many cases, the database switch never happens. A far more common scenario is changing where software runs. The core idea of cloud-nomad architecture is ensuring you can move the application without major changes. A key measure of success should be how easy it is to move your workloads to different cloud providers or back to your on-premises infrastructure.

Happy deployments!