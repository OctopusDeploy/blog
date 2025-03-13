---
title: Using Kubernetes to enhance CD best practices
description: This blog post is an excerpt from our free guide, Kubernetes delivery unlocked, which examines how you can use Kubernetes to enhance CD best practices.
author: liam.mackie@octopus.com
visibility: public
published: 2024-12-18-1400
metaImage: blog-kubernetes-unlocked-cd-best-practices-750x400.png
bannerImage: blog-kubernetes-unlocked-cd-best-practices-750x400.png
bannerImageAlt: Illustration of a box with Kubernetes logos, connected via a line to a screen displaying a ticked checklist of best practices, against a dark blue gradient background.
isFeatured: false
tags: 
  - DevOps
  - Kubernetes
  - Continuous Delivery
---

In the last 10 years, the deployment landscape has shifted significantly, but nothing has changed it more than Kubernetes. With the obvious benefits of Kubernetes comes yet another layer of complexity that developers need to overcome.

This post is an excerpt from our free guide, [Kubernetes delivery unlocked](https://oc.to/k8s-delivery-unlocked-excerpts-blog), which examines how you can use Kubernetes to enhance CD best practices. Although CD best practices have been around for a long time, there are plenty of opportunities to implement them in modern deployment practices. 

## CD best practices

The end goal of CD is to deploy quickly according to business needs. Its entire existence centers around reducing deployment friction so developers can focus on building good products without worrying about their work stopping critical fixes or other features. Although these best practices predate Kubernetes, they're all enhanced by Kubernetes in the modern software ecosystem.

### 1. Always be ready to deploy to production

The key tenet of CD is to always be ready to deploy to production. This is important because it decouples the ability to release with the readiness of any given feature. This lets you release new functionality based on the business's requirements, rather than the readiness of all features at any point in time. This also helps to fix bugs quickly, as you can make a fix and deploy it without stepping on the toes of developers working on features. Being able to deploy to production is, in the end, an exercise in risk management. You need to:

- Pick an easy-to-use branching strategy with short-lived branches
- Be able to deploy and test branches
- Run automated tests on both the main branch, as well as short-lived branches

By doing this, you build confidence in your main branch's ability to be pushed to production, as you've tested each change as a whole and in isolation. This also means that if a bug is being fixed, you can simply fix it and iterate on the main branch, confident that you can roll forward.

Kubernetes helps you be ready to deploy to production by providing easily isolated test environments. Often, you'll see a Kubernetes namespace used as a test environment, allowing a developer or team to simply deploy to an existing cluster. This removes the overhead of virtual machine (VM) management that would often prevent developers from running their software off their development environment.

### 2. Build once, deploy everywhere

Building once and deploying everywhere is important because it ensures there aren't surprises when you promote an artifact to production. What's signed off is exactly what's deployed. The artifact gets tested with automated tests, and you have confidence that the code approved by the various people involved in planning and building it is the same code that reaches production.

Building your code every time it changes is crucial to this process. You should build the main branch as well as short-lived feature branches. You should only use builds from short-lived branches for development and testing and should have pre-release versions to indicate this. Builds from the main branch should get automatically tested and successfully progress through all testing environments before reaching production.

Since Kubernetes uses containers, which are easily versioned and contain all necessary frameworks and libraries to run, it's well-suited to the task of running versioned build artifacts. This allows you to simply request Kubernetes to run a certain version, and you can be sure that it'll be the same, even between Kubernetes clusters. To further supercharge this functionality, many container registries let you enable immutable tags. This prevents you from overwriting a tag after it already exists. This ensures the code is always what you expect it to be.

### 3. Include all components

When it comes to software, the only way to ensure something will work is to test it, and the only way to test every code change effectively is through automation. This is also the case for the supporting components of your application. Manually updating components like your database schema, configuration, and infrastructure is risky and slow. Instead, your database schema and configuration should be in version control, and you should store your infrastructure as code. This concept is called Infrastructure as Code (IaC), and there are many tools that can handle this for you, including:

- Terraform
- CloudFormation
- ARM
- Bicep
- OpenTofu
- CrossPlane
- Pulumi

The deployment process should automatically run database schema upgrades, appropriately modify configuration files, and apply any changes to your infrastructure. Automatically running these processes against environments that closely resemble production gives confidence in the final push to production, and prevents bugs from surfacing when you least want them.

Since Kubernetes is a declarative system, much of the supporting infrastructure, such as disks, networking, and configuration, is abstracted from the underlying implementation. Your app simply needs 5GiB of storage, not a folder on a specific filesystem. This layer of abstraction makes it easy to change and validate your configuration consistently.

Kubernetes also provides tooling and operators to control external infrastructure, like Crossplane and AWS Controllers for Kubernetes (ACK). These controllers take declarative Kubernetes objects and create external resources like load balancers and databases. Since this infrastructure is declarative in nature, you can easily ensure that your infrastructure is consistent across environments.

### 4. Deploy to testing environments the same way as production

As we mentioned, the only way to ensure something will work as expected is to test it. This is important not only for code and components but also for the deployment process. The best way to test the production deployment process is to use the same process in test environments. This lets you find and fix issues earlier, leading to a shortened feedback loop and less wasted time.

Being confident in your deployment processes also helps when you have emergencies. Relying on a trusted, well-tested, automated process is much better than trying to improvise. Using an improvised process in a high-pressure situation makes you more likely to cause even more problems. Even if the fix works, you're unlikely to use it again, leading to a bunch of engineering time only used to fight fires, rather than improve the process.

Tools like Helm and Kustomize also let you template your deployments and configuration for each environment. Octopus, Codefresh, and ArgoCD are all tools that can deploy to Kubernetes consistently across environments, even dynamically.

## Conclusion 

The end goal of CD is to deploy quickly according to business needs. These best practices are all enhanced by Kubernetes in the modern software ecosystem, and understanding what the best practices are and how to apply them will help set you up for success. 

This post was an excerpt from our new guide, Kubernetes delivery unlocked, which helps you understand:

- How to use Kubernetes' architecture to achieve seamless, zero-downtime deployments
- How to define and build your deployment process using CD principles
- Common challenges faced by developers using Kubernetes and how to overcome them

[Download the free guide, Kubernetes delivery unlocked](https://oc.to/k8s-delivery-unlocked-excerpts-blog).

!include <related-content>

Happy deployments! 
