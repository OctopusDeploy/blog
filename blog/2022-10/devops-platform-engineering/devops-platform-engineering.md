---
title: DevOps and platform engineering
description: Find out where platform engineering fits into a DevOps process and why both DevOps and platform engineering can both help your organization attain high performance.
author: steve.fenton@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - tag
---

In this post, you'll find out where platform engineering fits into your broader software delivery process. You'll see where platform engineering works with a DevOps process and why both DevOps and platform engineering can both help your organization attain high performance.

The short description of DevOps is a reminder of how it evolved from a simple statement to a more complete picture. This is followed by an introduction to platform engineering, before they are pictured together.

## The quick version of DevOps

DevOps stems from the key idea of developers and ops working together. This had become difficult to do in many organizations because these teams had conflicting goals. Developers needed to ship more value, more frequently. The operations team needed to keep things stable. This sets up conditions where conflict is highly likely, if not guaranteed.

To remedy this dev and ops started working more collaboratively and this turned out to be a more effective way to delivery high quality software.

Over the course of ten years, the vague value statement of "developers and ops working together" grew into a very well-defined set of capabilities, thanks to extensive research by Puppet and DORA. The DevOps "structural equation model" maps the capabilities and relationships found in the research.

![The 2022 DevOps structural equation model](structural-equation-model-2022.png)

This model is useful for teams looking for improvement opportunities and organizations seeking the benefits other organizations have attained with DevOps. You may have seen an older version of this diagram, which had fewer boxes.

As you can see, the 2022 model is packed with ideas for specific capabilities you can adopt to become *more DevOps*. If you feel overwhelmed, [how to start using Continuous Delivery](https://octopus.com/devops/continuous-delivery/how-to-start-using-continuous-delivery/) in our [DevOps Engineer's handbook] is a good place to start.

The crucial insight in this model is the importance of culture to both the technical performance of your organization and its performance against commercial and non-commercial goals.

In 2022, DevOps has grown to mean:

- Developers and ops working together
- A well-defined set of technical and non-technical capabilities
- Assessing your success using whole-system measures

If you've been around long enough, you might notice that many of the changes encouraged by DevOps look a lot like how systems were developed before dev and ops silos were created. Specialist teams were created for a reason, so as we back-up and try another path, we should ensure we don't repeat past mistakes. The problem of scale and specialization still exist, so how do we overcome them in a more healthy way?

Enter the platform engineering team.

## Platform engineering

Despite many new teams and job titles springing up around DevOps, the platform engineering team is, perhaps, the most aligned to the mindset and objectives of DevOps.

A platform team treats the developers as customers, smoothing a set of pathways that help developers spend less energy on non-programming tasks. They don't handle tickets, such as "create a new test database", but instead provide usable self-service mechanisms for teams to create the database easily themselves.

Platform teams work with development teams to create one or more *golden pathways*, which represent a supported set of technology choices. These pathways don't prevent teams using something else, but they encourage more alignment without enforcing standard choices on other teams.

A critical part of platform engineering is how they tread developers as customers, solving their problems and reducing friction while advocating the adoption of aligned technology choices.

As a developer, by choosing to use a golden pathway, you get many of your needs for free. This accelerates your work and gives you a support channel when things go wrong.

Platform engineering makes lots of things easy:

- Build pipelines
- Test and production environments
- Automated deployments
- Test frameworks
- Logging and monitoring
- Security features

If you are using DevOps and you are scaling up, platform engineering reduces the operations burden on all your development teams, which means you need fewer of these hard-to-find platform engineers overall.

Platform engineering helps your organization scale its software delivery without losing some of the best small-team benefits.

## DevOps and platform engineering

As you can see, platform engineering complements, rather than competes, with DevOps. To provide further proof of this positive relationship, the [Puppet State of DevOps report](https://puppet.com/resources/report/2021-state-of-devops-report) found that DevOps high-performers are more likely to have a platform engineering team than low performers.

![DevOps teams using platform engineering, table follows](devops-platform-engineering-performance.png)

| Category | % with platform engineering |
|----------|----------------------------:|
| Low      | 8%                          |
| Mid      | 25%                         |
| High     | 48%                         |

Platform engineering alone doesn't provide a whole organization view of performance. The DevOps structural equation model shows us capabilities for leadership, management, culture, and product that are outside of the scope of a platform team.

Used together, platform engineering is an excellent tool for scaling software delivery.

DevOps:

- Measure the performance of the whole system
- Shorten and amplify feedback loops
- Create a culture of continuous learning and improvement

Platform engineering:

- Smooth the development experience
- Create tools and workflows that enable self-service
- Make it easy for developers to achieve system quality attributes (such as performance, observability, and security)

## Conclusion

As you grow your software delivery team, the complexity that comes with scale will need to be managed. Some organizations will limit complexity by limiting how much freedom teams have to make choices, but platform engineering provides a mechanism that manages complexity while preserving development team autonomy.

Happy deployments!

## Further reading

- Building Secure and Reliable Systems - Heather Adkins, et al
- [Octopus DevOps Engineer's Handbook](https://octopus.com/devops/)