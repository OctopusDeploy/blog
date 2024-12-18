---
title: Choosing the right CD tool for your Kubernetes deployments
description: This post is an excerpt from our free guide, Kubernetes delivery unlocked. It examines the main categories of Continuous Delivery tools.
author: liam.mackie@octopus.com
visibility: public
published: 2025-01-22-1400
metaImage: unlocked-choosing-the-right-cd-tools-750x400.png
bannerImage: unlocked-choosing-the-right-cd-tools-750x400.png
bannerImageAlt: Illustration of tech toolbox with Kubernetes logo, surrounded by icons for GitHub, Docker, Kubernetes, AWS, and Octopus Deploy, connected by a CD pipeline.
isFeatured: false
tags: 
  - DevOps
  - Kubernetes
---

In the last 10 years, the deployment landscape has shifted significantly, but nothing has changed it more than Kubernetes. With the obvious benefits of Kubernetes comes yet another layer of complexity that developers need to overcome.

This post is an excerpt from our free guide, [Kubernetes delivery unlocked](https://octopus.com/whitepapers/kubernetes-delivery-unlocked). It examines the main categories of Continuous Delivery (CD) tools, each with its own levels of abstraction and complexity, to help you choose the right tool for your Kubernetes deployments. 

## Choosing your CD tooling

There are 3 main categories of CD tools that you can choose from:

- Combined CI/CD tools
- GitOps tools
- Dedicated CD tooling

## Combined CI/CD tools

The simplest tools to use are often the ones integrated with your CI tooling. As an example, GitHub Actions has steps published by Azure to deploy to Kubernetes. While well integrated, this approach can often only be scaled for simple workflows. To promote an artifact to an environment, you must reinvent the wheel and implement it in GitHub Actions. Soon, you (or your development team) will develop actions to deploy your code, rather than the code itself.

## GitOps tools

Due to Kubernetes' declarative nature, tools that rely on GitOps principles are widely available. ArgoCD is one of the leading tools to implement GitOps. If you have a more complex deployment pipeline but love ArgoCD, Codefresh uses ArgoCD as a base to iterate and improve on, adding things like environment progression and an overview spanning many clusters. Though ArgoCD is the most recognizable name in the space, dedicated CD tooling like Octopus Deploy, and even combined CI/CD tooling like GitLab, has support for GitOps, often allowing code-first configuration that you can commit to version control.

## Dedicated CD tooling

Dedicated CD tooling is characterized by CD concepts being best-of-breed. Deployments, releases, tenants, and environments are all supported and managed separately. This lets you build a deployment pipeline that can evolve as your software does.

These dedicated tools, like Octopus Deploy, are flexible enough to fine-tune your deployment process depending on your exact requirements and will have built-in support for Kubernetes, along with any other system you'll ever need to deploy to.

These tools often include user access control, dashboards, metrics, history, and logs. They aim to be your one-stop shop for the deployment process. Tools like Octopus Deploy are usually structured as a pipeline. Your input is a set of build artifacts, and the output is your software deployed to an environment. They let you intervene if there's a change that needs QA or approval. They allow for automatic environment and tenant progression and help automate your entire environment's process.

## Conclusion 

Your tooling choice ends up being less about the tooling, and more about your needs. If you don't have a complex pipeline, you won't get much value out of a dedicated CD tool, which usually has licensing costs. If you have a complex pipeline, though, these licensing costs will usually be outweighed by the time and effort you save your engineering team.

This post was an excerpt from our new guide, Kubernetes delivery unlocked, which helps you understand:

- How to use Kubernetes' architecture to achieve seamless, zero-downtime deployments
- How to define and build your deployment process using CD principles
- Common challenges faced by developers using Kubernetes and how to overcome them

[Download the free guide, Kubernetes delivery unlocked](https://octopus.com/whitepapers/kubernetes-delivery-unlocked).

Happy deployments! 