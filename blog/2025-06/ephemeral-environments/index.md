---
title: Help shape Ephemeral Environments
description: Learn about Ephemeral Environments, coming soon to Octopus, and help us shape the feature.
author: harriet.alexander@octopus.com
visibility: public
published: 2025-06-23-1400
metaImage: 
bannerImage: 
bannerImageAlt: 
isFeatured: false
tags: 
  - Product
  - Environment Promotion
---

In the fast-paced world of software development, few phrases provoke as much frustration as “it works on my machine”. It’s the ultimate debugging roadblock. Your code works flawlessly in your local environment, but everything breaks when it merges. But what if there was a way to ensure your code wasn’t just “working on your machine” but on an environment that mirrors your production environment before you merge?

In this post, I introduce our solution—Ephemeral Environments.

## What are Ephemeral Environments?

We've been hard at work developing Ephemeral Environments, tailored for modern development workflows. These environments are automatically generated and linked to feature branches created through pull requests (PRs). They offer a temporary space for testing code changes without affecting the main lifecycle environments. 

The primary goal is to **shift left** in the development process: identifying issues earlier when they are easier and less costly to address. This then minimizes surprises later in the workflow. 

## How Ephemeral Environments will work

- Feature branch-based: When you create a pull request (PR) from your feature branch, the ephemeral environment automatically spins up.
- Temporary by design: After you close or merge the PR, the environment spins down in a hassle-free manner.
- Designed for early feedback: The feature will provide developers and collaborators with early integration testing, UI validation, and fast feedback loops.

## Why you should get excited

Catching issues earlier in the development lifecycle leads to better software, faster delivery, and happier teams. Here’s why we’re excited about Ephemeral Environments—and why we think you will be, too:

- Fewer environment bottlenecks: Don’t waste time waiting for staging slots or shared resources. Every feature branch gets its own environment.
- Increased confidence in merges: Know your code works before merging into the main branch, so there are fewer surprises and regressions.
- Seamless feedback loops: Ephemeral Environments are dynamic and easy to share. This increases collaboration across teams and stakeholders.

## Have your say and get early access to Ephemeral Environments

We’re building this feature to make pull request workflows smarter, faster, and more reliable— but we'd love your feedback.

Whether you want to sign up for a product demo, participate in our alpha testing, or simply get notified when early access is available, we want to hear from you.

### What’s in it for you?

- Get early access: Get access to the ephemeral environment feature before it’s widely available.
- Influence design: Your feedback will directly shape how we refine the feature, ensuring it meets real-world workflows.

## How to sign up

[Register your interest via our form](https://admin.typeform.com/form/ZOia9Aje/create?block=9ded49c0-1887-400a-a9ba-5e2ae8aab68d) to have your say.

We're hoping to release ephemeral environments in all instances later this year.

Happy deployments!
