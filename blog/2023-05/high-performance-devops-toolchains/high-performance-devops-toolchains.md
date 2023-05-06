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

One of the many interesting sections in this year's report looked at DevOps-related technologies. Inspired by this insight, we take a deep dive into how to build high-performance DevOps toolchains.

## DevOps tools and toolchains

You will have heard that DevOps isn't just about tools. There are several critical elements of high-performance software delivery, such as:

- Transformational leadership
- Lean product management
- Technical practices
- Organizational culture

Even your change approval process impacts your software delivery and, ultimately, your success as an organization. Despite being about *more* than tools, your DevOps toolchain remains crucial.

When you select great tools and arrange them well, you create a cohesive DevOps toolchain. You can deploy frequently and reliably with a high degree of confidence. If you get it wrong, the tools get in the way and slow you down.

A set of DevOps tools becomes a *toolchain* when it covers the whole Continuous Delivery pipeline:

- Work item management
- Source control
- Build and test automation
- Security testing 
- Artifact repositories
- Infrastructure as code and configuration management 
- Deployment automation
- Monitoring and observability
- Incident management

You'll be spoiled for choice when you look at [DevOps tools](https://octopus.com/devops/continuous-delivery/continuous-delivery-tools/). A great toolchain is a union of strong tools and simple integration design. Some tools, or combinations of tools, will make your life more difficult.

## DevOps tool research

The [2023 State of CD report](https://cd.foundation/reports/) looked at this in depth. The researchers looked at the impact of tools and technologies on:

- Lead time for changes
- Deployment frequency
- Time to restore service

The report found that high performers use more tools than low performers. Almost 45% of low performers were using a single tool and deploying less frequently than once a month. High performers were 2x more likely to be using 10+ tools than a single tool. Teams were significantly less likely to be low performers if they used CI/CD tools to automate builds and deployments.

> Those who use CI/CD tools are significantly less likely to be low performers than those who do not

To achieve similar results, you need to assemble the correct tools in the right way. Teams who self-hosted multiple CI/CD tools tended to be lower performers as interoperability issues and manual handling slow the deployment pipeline.

## High-performance DevOps toolchains

There are 4 steps to building high-performance DevOps toolchains.

1. Select best-in-class tools
2. Prefer managed solutions
3. Keep workloads clean
4. Limit tool dependencies

### Select best-in-class tools

For the critical elements of your toolchain, you need best-in-class tools. In particular, you need the best tools you can get for your build server, deployment automation, and monitoring. Specilized tools will be carefully designed to simplify complex workloads.

It is often *possible* to run workloads on a general purpose platform. You may even save some money by adding workloads to an existing tool. However, you tend to pay in other ways, such as longer lead times and lower deployment frequency.

Low performance in software delivery is always more expensive than the tools.

### Prefer managed solutions

Another consideration is balancing the number of tools you self-host. Hosting a complete toolchain is no small endeavor and the work involved in keeping it updated, stable, and responsive shouldn't be taken lightly.

You can use managed services to reduce the operational burden of your tools. Only self-host where you have a strong reason to do so.

### Keep workloads clean

Don't clutter a workload with too many tools. If you need to cobble together several tools to perform one workload, such as building a software version, it will slow you down. Subtle issues will routinely crop up and need to be fixed, moving focus away from delivering valuable software.

### Limit tool dependencies

Keep the dependencies between tools minimal and simple. You don't want to spend time investigating interoperability issues as it stops you doing more valuable work.

Control should be passed along the pipeline, with tools only being aware of tools that precede or follow them.

For example, consider this common Continuous Delivery pipleine:

1. Version control (Git)
2. Build server (Jenkins)
3. Artifact repository (Octopus)
4. Deployment automation (Octopus)
5. Monitoring and alerting (DataDog)
6. On call and incident management (PagerDuty)

You can replace Jenkins with a different build server as long as you make sure the output is pushed to your artifact repository in the same format. None of the other tools are impacted, so this is a well-designed toolchain.

If you had to update your deployment process or version control, it would suggest they are too tightly coupled to Jenkins.

## Conclusion

A strong DevOps toolchain is a mix of great tools and good design. Combining tools into a cohesive toolchain improves performance, while poor tools, the overheads of self-hosting, and interoperability issues cause slow downs.

Keep in mind the 4 steps to a great toolchain:

1. Select best-in-class tools
2. Prefer managed solutions
3. Keep workloads clean
4. Limit tool dependencies

Happy deployments!
