---
title: Inside DevOps with Anton Zagrebelny from Stigg
description: A series where we share lessons learned from those on the frontlines of DevOps. This post features Anton Zagrebelny from Stigg.
author: joanna.wyganowska@octopus.com
visibility: public
published: 2023-11-27-1400
metaImage: blogimage-insidedevops-antonz-stigg-2023.png
bannerImage: blogimage-insidedevops-antonz-stigg-2023.png
bannerImageAlt: Photo of Anton Zagrebelny
isFeatured: false
tags: 
  - DevOps
  - Inside DevOps
---

This post is the next in our [Inside DevOps series](https://octopus.com/blog/tag/Inside%20DevOps), where we share lessons learned from those on the frontlines of DevOps.  

Hear from [Anton Zagrebelny](https://www.linkedin.com/in/anton-zagrebelny/), co-founder and CTO at Stigg, a headless pricing and packaging platform provider.

**What is DevOps to you? How do you define it?**

*Anton:* DevOps, to me, is the practice of materializing a software product and ensuring it runs effectively in terms of both scale and cost. It's not just about automating deployments or monitoring infrastructure. It's the entire lifecycle of a software product from development to production.

To give you an example from [Stigg](https://www.stigg.io/), we invested in a DevOps culture from the get-go. Within the first 2 weeks, our engineering team had already automated the build, test, and deployment processes, and set up Infrastructure as Code (IaC) methodology. This let us iterate on our product rapidly, deploying new versions dozens of times a day and modifying our CloudFormation stacks on the fly without breaking a sweat. Given that our production environment is mission-critical, we opted for a serverless approach to ensure scalability. Adopting best practices in IaC and automation was crucial for us to support this kind of agility. 

**How did your DevOps journey start?**

*Anton:* My DevOps journey kicked off when I founded a one-stop-shop software services company. We worked with a diverse set of clients, each with unique needs, tech stacks, and even different cloud providers. This complexity led us to invest heavily in Kubernetes and Terraform to create multi-tenant, multi-cloud deployment workflows that automated most of the work.

**What’s the most challenging part of DevOps?**

*Anton:* The most challenging part is staying up-to-date with both internal and external changes. Internally, company needs, products, and personnel are always evolving. Externally, the tech landscape is constantly shifting. Balancing these dynamics and adapting your DevOps practices accordingly separates a good DevOps engineer from a great one.

**What’s the most rewarding part of DevOps?**

*Anton:* The most rewarding part for me is seeing a fully automated process that starts with a simple code push to a Git repository and ends with both infrastructure provisioning and software deployment. It's like watching a well-oiled machine at work.

**What are some DevOps best practices you and your organization have implemented?**

*Anton:* At Stigg, we manage all our infrastructure using AWS CDK and CloudFormation, version-controlled in Git alongside our back-end code. Post-push, GitHub Actions takes over for build and test, and then Octopus Deploy handles the deployment to various environments. We even wrote a [blog post](https://www.stigg.io/blog-posts/how-we-built-our-ci-cd-pipeline-for-velocity-and-quality) about our CI/CD pipeline and the principles behind it.

**What’s the biggest challenge Octopus has helped you with?**

*Anton:* We're using Octopus Deploy for managing our complex, multi-environment deployments. It allows us to design intricate deployment processes for infrastructure and application code, and trigger end-to-end tests. And it's great that we can roll back changes with a click if anything goes wrong.

**What advice would you give someone just starting their DevOps journey?**

*Anton:* Start small and get the basics down first. Familiarize yourself with Docker, understand containerization, get comfortable with bash scripting, and pick up some networking fundamentals. These foundational skills will serve you well as you dive deeper into the DevOps world.

**What DevOps book would you recommend reading?**

*Anton:* I'd recommend [The DevOps 2.0 Toolkit](https://github.com/mpattanaik7/docker-books/blob/master/The%20DevOps%202.0%20Toolkit.pdf) by Victor Farcic. Whether you're a beginner or an expert, this book offers valuable insights into the evolving landscape of DevOps.

**What's one thing about you that might surprise us?**

*Anton:* By age 18, I’d spent 10 years learning 6 different types of martial arts. It's a discipline that has taught me a lot about focus and perseverance, which I find incredibly useful in my work.

:::hint
If you’d like to be featured in our series, [Inside DevOps](https://octopus.com/blog/tag/Inside%20DevOps), please reach out to [Joanna on LinkedIn](https://www.linkedin.com/in/joannawyganowska/) to set up a time for a quick chat.
:::

Happy deployments!
