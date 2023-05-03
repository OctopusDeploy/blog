---
title: Creating a high-performance DevOps toolchain
description: Discover the elements of a high-performance DevOps toolchain and the research that backs it up.
author: steve.fenton@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 
isFeatured: false
tags: 
  - DevOps
---

The [2023 State of CD report](https://cd.foundation/reports/) was released this week. The report is commissioned by the Continuous Delivery Foundation and authored by SlashData. The State of CD report looks into the adoption, practices, and performance of teams with a focus on DevOps and Continuous Delivery.

One of the many interesting sections in this year's report looked at DevOps-related technologies. Inspired by this insight, we take a deep dive into high-performance DevOps toolchains.

## DevOps toolchains

DevOps isn't just about tools. Transformational leadership, Lean product management, technical practices, and organizational culture are critical elements of high-performance software delivery. Despite this, your DevOps toolchain can make or break your DevOps efforts.

A cohesive DevOps toolchain involves more than selecting great tools. You also need to consider interoperability and apply some design to how you assemble them. The technical skills you apply to your code will help you create a solid toolchain.

*DevOps tools* is a term that covers workloads like:

- Work item management
- Source control
- Build and test automation
- Security testing 
- Artifact repositories
- Infrastructure as code and configuration management 
- Deployment automation
- Monitoring and observability
- Incident management

There are many great options, but the selection and assembly process needs some thought. The toolchain should boost your performance, but some tools, or combinations of tools, will make your life more difficult.

The [2023 State of CD report](https://cd.foundation/reports/) looked at this in depth, finding:

- CI/CD tools help you achieve high performance
- Interoperability issues can emerge from your toolchain

READ PAGE 34 and grok what this means!!!

## The importance of CI/CD tools

The research found that developers using CI/CD tools deploy more often, have shorter lead times, and restore services faster when there's an incident.

> Those who use CI/CD tools are significantly less likely to be low performers than those who do not

## Integrating best-in-class tools

There is no available platform that handles the complete DevOps workflow. If there was, it would likely be weak in many areas. The concept of an all-in-one platform usually features a strong source control and build server story, but it gets weaker as you move from the central features. In other words, it's not all-in-one, it's many-but-not-all.

A great DevOps toolchain selects best-in-class tools for crucial workflows like source control, builds, deployments, and monitoring. 

The research backs up the benefits of a multi-technology approach. Developers with a single-technology approach were more likely to be low performers, whereas teams with 10 or more tools were more likely to be high performers.

The critical thing to manage when you build a DevOps toolchain is the integrations. You need to minimize dependencies between tools and ensure selected tools don't unduly limit your options.

There are three factors that impact interoperability:

1. The selected tool's integration points
2. How well you minimize dependencies
3. The design of your integrations

You should consider interoperability during your selection phase as this is difficult to overcome after you've purchased licenses.

## Key tool selection considerations

For critical areas, choose best-in-class tools.

Arrange the toolchain as a sequence where possible, this reduces the number of tools each tool needs to know about. For example, if your build server knows about source control and your deployment tool, it should mean other tools don't need to depend on them. It should be possible to change your source control provider with no disruption to tools that lay beyond the build server.

## Conclusion

The research shows that software delivery performance is improved when developers use the right combination of DevOps technologies. Teams who limit developers to too few tools are most likely to underperform.

To avoid the pitfalls of integrating many tools, careful tool selection and strong integration design will ensure performance gets a sustainable increase.

Happy deployments!
