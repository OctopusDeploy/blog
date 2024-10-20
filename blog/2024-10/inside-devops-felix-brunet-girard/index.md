---
title: "Inside DevOps with Félix Brunet Girard from TELUS Health"
description: A series where we share lessons learned from those on the frontlines of DevOps. This post features Félix Brunet Girard, Lead DevOps Specialist with TELUS Health.
author: joanna.wyganowska@octopus.com
visibility: public
published: 2024-10-24-1400
metaImage: img-blog-inside-devops-greenblue-750x400-2024-x2.png
bannerImage: img-blog-inside-devops-greenblue-750x400-2024-x2.png
bannerImageAlt: Photo of Félix Brunet Girard
isFeatured: false
tags: 
  - DevOps
  - Inside DevOps
---

This post is the next in our [Inside DevOps series](https://octopus.com/blog/tag/Inside%20DevOps), where we share lessons learned from those on the frontlines of DevOps.  

Hear from [Félix Brunet Girard](https://www.linkedin.com/in/felixbrunetgirard/), Lead DevOps Specialist with TELUS Health (TELUS Santé), a global health and wellbeing provider.


**What is DevOps to you? How do you define it?**

*Félix:* For me, it's a culture first. The goal is to help development and operations teams align. It’s also a responsibility change because developers need to be aware of how they’re testing and how their application gets deployed. No longer can you just compile code and throw it to another team. You need to understand the impact of what you’re doing. Developers need to understand the security implications and how to test for security so the deployment process is always reliable and safe. 

The goal of DevOps is also to improve agility and the feedback loop. The tooling is a part of it, but that’s just what we use to obtain the value we want, as long as teams stay close to each other. But DevOps isn’t simple – if you keep the same organization topology and roles, you can’t expect to reap the benefits of DevOps.


**How did your DevOps journey start?**

*Félix:* My experience with DevOps started when I was at Croesus. Our VP of Engineering often talked about agility and DevOps. He wanted the DevOps team to be able to create CI/CD pipelines. I came from a developer background, so didn’t know what that even meant. 

I developed my own build tools to compile code faster and some CI for testing. But this was maybe 50% of my time. I knew I needed to make this a bigger part of my career if I wanted to truly gain DevOps knowledge and proficiency with DevOps tools.

Currently, I’m with TELUS Health, working both with on-prem and cloud applications, so there are different technologies to learn. I enjoy implementing new tools to help meet the needs of our developers and architects.

**What are some DevOps best practices your organization has implemented?**

*Félix:* We’ve implemented a “golden pipeline”, which is a reusable template for building and deploying applications. With the use of Octopus for CD, we’ve standardized our deployment processes and made them repeatable, which saves us time and provides stability.

We have also implemented Azure Monitor / Application Insights for monitoring app performance.

**What’s the most challenging part of DevOps?**

*Félix:* There are many different cloud providers and tools, so you can’t learn everything. Also, there’s significant toil due to the integrations between tools that require a lot of scripting. The challenging part is also that DevOps teams can have too much responsibility and become a new bottleneck.
 
An example is when the dev team doesn’t own their build/release pipelines, and it can take a lot of energy to handle multiple requests at the same time. Planning in DevOps is also hard, since there’s often urgency related to deployments and a lot of fire-fighting.

**What’s the biggest challenge Octopus has helped you with?**

*Félix:* With Octopus, you can have a simple deployment flow but with good customization. This is very important to us as our solution is very large with lots of components. Various clients have different components, so we use target roles and tags, for example. As everything is reusable in Octopus, with good variables handling, we shorten the time to deploy and make it repeatable. 

With Octopus, we can also deploy anywhere (on-prem, cloud, AKS, etc) using one tool. We use Octopus for multiple lines of business, so this standardization makes troubleshooting easier. It also helps us reduce the complexity of having multiple release pipelines.

Octopus’s UI is very friendly, so different people with different skills or roles can use it.

**What’s the most rewarding part of DevOps?**

*Félix:* I think helping others achieve their goals and helping the organization with big issues. It’s very rewarding to help someone that was having issues, and you don’t always get that as a developer. Also, you learn new stuff almost every week, as you need to understand the technology stack, which always evolves.

**What advice would you give to those just starting their DevOps journey?**

*Félix:* Be curious and try new things. Start small and improve step-by-step.

**What DevOps book would you recommend reading?**

*Félix:* My top 4 books are:

- *[Continuous Delivery: Reliable Software Releases through Build, Test, and Deployment Automation](https://octopus.com/devops/reading-list/#continuous-delivery)* by Jez Humble and David Farley.
- *[The DevOps Handbook: How to Create World-Class Agility, Reliability, & Security in Technology Organizations](https://octopus.com/devops/reading-list/#the-devops-handbook)* by Gene Kim, Jez Humble, et al.
- *[Effective DevOps: Building a Culture of Collaboration, Affinity, and Tooling at Scale](https://www.goodreads.com/book/show/25602743-effective-devops)* by Jennifer Davis and Ryn Daniels
- *[Team Topologies: Organizing Business and Technology Teams for Fast Flow](https://octopus.com/devops/reading-list/#team-topologies)* by Matthew Skelto

**What’s one thing about you that might surprise us?**

*Félix:* A fun fact about me is that I’m a third-degree black belt in Chinese Kenpo.

**Thank you for sharing your DevOps insights with us, Félix; it’s much appreciated.**

:::hint
If you’d like to be featured in our series, [Inside DevOps](https://octopus.com/blog/tag/Inside%20DevOps), please reach out to [Joanna on LinkedIn](https://www.linkedin.com/in/joannawyganowska/) to set up a time for a quick chat.
:::