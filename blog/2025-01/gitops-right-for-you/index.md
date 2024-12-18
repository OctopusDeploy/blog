---
title: "Is GitOps right for you?" 
description: This post is an excerpt from our free guide, Kubernetes delivery unlocked. This section examines GitOps and its implications for setting up pipelines.
author: liam.mackie@octopus.com
visibility: public
published: 2025-01-29-1400
metaImage: blog-kubernetes-unlocked-gitops-750x400.png
bannerImage: blog-kubernetes-unlocked-gitops-750x400.png
bannerImageAlt: An illustration of a box that's being unlocked, containing Kubernetes logos, set against a dark blue gradient background.
isFeatured: false
tags: 
  - DevOps
  - Kubernetes
  - GitOps
---

In the last 10 years, the deployment landscape has shifted significantly, but nothing has changed it more than Kubernetes. With the obvious benefits of Kubernetes comes yet another layer of complexity that developers need to overcome.

This post is an excerpt from our free guide, [Kubernetes delivery unlocked](https://octopus.com/whitepapers/kubernetes-delivery-unlocked). This section examines GitOps and its implications for setting up pipelines, and the benefits and challenges of opting for a GitOps approach. 

## What is GitOps, and what does it mean for setting up your pipelines? 

If you've been learning more about Kubernetes, you've likely seen the term GitOps. But what is it, and what does it mean for setting up your pipelines?

GitOps is a set of principles that build on the best practices of CD and DevOps. With the advent of Kubernetes and other tools, complex systems can be defined declaratively. These declarative systems are very easily stored in Git, letting developers use familiar workflows and tooling for both development and deployment.

The core of GitOps is to commit the desired state of your application to a repository. GitOps tooling then watches this repository. When you update the repository, the tools continuously reconcile the target system to ensure that the state in the repository is reflected on the target system.

The 4 main principles of GitOps, as defined by Open GitOps, are:

1. Declarative
2. Versioned and immutable
3. Pulled automatically
4. Continuously reconciled

This leads to a remarkably resilient system, where you can tell what version is running based on the commits to the environment's branch/repository. To promote between environments, you can use merge/pull requests.

## Benefits of GitOps

There are a number of benefits to using a GitOps approach.

### Better audibility

Since the environment's history is stored in Git, you can view its entire history using Git. This shows the differences between versions, the person who committed them, and who approved them.

### Improved consistency

When GitOps tools reconcile a system's state, they remove any changes made externally. This ensures that your environments are consistent with Git. When your environments are consistent with Git and have no manual changes, you can be more certain in the deployment process. This tightly aligns with CD best practices.

### Increased speed

Most developers can interface quickly with systems based on Git. Making a change and pushing it to Git is a quick and easy process, making deploying to an environment a small, simple task. Since Git is made to be collaborative, multiple people can work on a single change, and you can resolve any conflicts easily using built-in tools.

## Challenges of GitOps

While GitOps is an excellent choice when operating declarative systems, it can have some challenges. These challenges are some of the more common pitfalls when adopting GitOps.

Though they aren't insurmountable, you should consider whether GitOps is the best option for your use case.

### Learning curve

For engineering teams unfamiliar with Kubernetes and declarative infrastructure, GitOps can present a steep learning curve. Without tooling and practice, GitOps can feel very restrictive. For example, if your team is used to managing infrastructure in an ad-hoc manner, they may have some teething pain when they're required to instead commit all desired states to Git. Much like learning to touch-type, there's sometimes a small drop in productivity for engineers while they get used to GitOps. After they learn, however, mistakes are less frequent, and engineers tend to be faster overall.

### Secrets

Git, despite all its benefits, is not a secure system. After you push a commit, it's almost impossible to remove it - especially if someone else has pulled or forked your repository. This is a benefit in most cases (see: better auditability), but if someone accidentally commits the API key to your AWS account in cleartext, you will have to work quickly to rotate it.
There are many secret providers available in Kubernetes. These providers let you connect to external systems to retrieve secret data. This lets you reference your secrets inside your Git repository, while keeping the actual data out of it. These external secret providers are in general, a best practice to use regardless of your adoption of GitOps practices.

### Environments

GitOps is centered around deploying a single application instance to a single cluster. Tools and workflows help manage multiple application instances, but these introduce complexity to what is, at its core, a simple system. These often require multiple repositories, leading to sprawl and difficulty finding what defines a resource you're trying to modify.


## Conclusion 

GitOps is a popular set of principles that build on the best practices of CD and DevOps. While it's a popular choice, there are some common pitfalls to keep in mind as you set up your Kubernetes pipeline. 

This post was an excerpt from our new guide, Kubernetes delivery unlocked, which helps you understand:

- How to use Kubernetes' architecture to achieve seamless, zero-downtime deployments
- How to define and build your deployment process using CD principles
- Common challenges faced by developers using Kubernetes and how to overcome them

[Download the free guide, Kubernetes delivery unlocked](https://octopus.com/whitepapers/kubernetes-delivery-unlocked).

Happy deployments! 