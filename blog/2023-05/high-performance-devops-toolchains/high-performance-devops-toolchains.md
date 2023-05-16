---
title: Creating a high-performance DevOps toolchain
description: Discover the elements of a high-performance DevOps toolchain and the research that backs it up.
author: steve.fenton@octopus.com
visibility: public
published: 2022-05-17-1400
metaImage: blogimage-highperformancedevopstoolchain-2023.png
bannerImage: blogimage-highperformancedevopstoolchain-2023.png
bannerImageAlt: Stylized image of two people doing maintenance work on a DevOps infinity symbol with a cog and timer at its center.
isFeatured: false
tags:
  - DevOps
  - Continuous Delivery
---

Last week, the [2023 State of CD report](https://cd.foundation/reports/) was released. The Continuous Delivery Foundation commission the report and it's authored by SlashData. The State of CD report looks at the adoption, practices, and performance of teams using DevOps and Continuous Delivery.

One of the many interesting sections in this year's report looked at DevOps-related technologies. Inspired by this insight, we take a deep dive into how to build high-performance DevOps toolchains.

## DevOps tools and toolchains

DevOps isn't just about tools. There are several other critical elements of high-performance software delivery:

- Transformational leadership
- Lean product management
- Technical practices
- Organizational culture

Even your change approval process impacts your performance and, ultimately, your success as an organization. Despite being about *more* than tools, your DevOps toolchain remains crucial.

You create a cohesive DevOps toolchain by selecting great tools and arranging them well. You can deploy frequently and reliably with a high degree of confidence. If you get it wrong, the tools get in the way and slow you down.

A set of DevOps tools becomes a *toolchain* when it covers the whole Continuous Delivery pipeline:

- Work item management
- Source control
- Build and test automation
- Security testing 
- Artifact repositories
- Infrastructure as Code and configuration management 
- Deployment automation
- Monitoring and observability
- Incident management

You're spoiled for choice when you look at [DevOps tools](https://octopus.com/devops/continuous-delivery/continuous-delivery-tools/). A great toolchain is a union of solid tools and simple integration design. Some tools, or combinations of tools, will make your life more difficult.

## DevOps tool research

The [2023 State of CD report](https://cd.foundation/reports/) looked at tools in depth. The researchers looked at the impact of tools and technologies on:

- Lead time for changes
- Deployment frequency
- Time to restore service

The report found that high performers use more tools than low performers. Almost 45% of low performers used a single tool and deployed less frequently than once a month. High performers were 2x more likely to be using 10+ tools than a single tool. Teams were significantly less likely to be low performers if they used CI/CD tools to automate builds and deployments.

> Those who use CI/CD tools are significantly less likely to be low performers than those who do not

To achieve similar results, you need to assemble the correct tools in the right way. Teams who self-hosted multiple CI/CD tools tended to be lower performers. Interoperability issues and manual handling are common causes of deployment pipeline slowdowns.

## High-performance DevOps toolchains

There are 4 steps to building high-performance DevOps toolchains:

1. Select best-in-class tools
2. Prefer managed solutions
3. Keep workloads clean
4. Limit tool dependencies

### Select best-in-class tools

For the critical elements of your toolchain, you need best-in-class tools. In particular, you need the best tools you can get for your build server, deployment automation, and monitoring. Specialized tools are carefully designed to simplify complex workloads.

It's often *possible* to run workloads on a general-purpose platform. You may save some money by adding workloads to an existing tool. But, you tend to pay in other ways, such as longer lead times and lower deployment frequency.

Low performance in software delivery can be more expensive than licenses for tools.

### Prefer managed solutions

You need to limit the number of tools you self-host. Hosting a complete toolchain is no small endeavor. You shouldn't take lightly the work involved in keeping it updated, stable, and responsive.

You can use managed services to reduce the operational burden of your tools. Only self-host where you have a strong reason to do so.

### Keep workloads clean

Don't clutter a workload with too many tools. It will slow you down if you need to cobble together several tools to perform one workload, such as building a software version. Subtle issues will routinely crop up that you must fix, which moves your focus away from delivering valuable software.

### Limit tool dependencies

Keep the dependencies between tools minimal and simple. You don't want to investigate interoperability issues as it stops you from doing more valuable work.

Control should flow along the pipeline, with tools only being aware of tools that precede or follow them.

For example, consider this common Continuous Delivery pipeline:

1. Version control (Git)
2. Build server (Jenkins)
3. Artifact repository (Octopus)
4. Deployment automation (Octopus)
5. Monitoring and alerting (DataDog)
6. On-call and incident management (PagerDuty)

You can replace Jenkins with a different build server if you ensure you push the output to your artifact repository in the same format. You don't impact the other tools, so this is a well-designed toolchain.

If you had to update your deployment process or version control, it would suggest they're too tightly coupled to Jenkins.

## Conclusion

A strong DevOps toolchain is a mix of great tools and sound design. Combining tools into a cohesive toolchain improves performance. However, poor tools, self-hosting overheads, and interoperability issues cause slowdowns.

Keep in mind the 4 steps to a great toolchain:

1. Select best-in-class tools
2. Prefer managed solutions
3. Keep workloads clean
4. Limit tool dependencies

Happy deployments!