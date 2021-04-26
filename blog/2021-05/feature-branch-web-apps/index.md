---
title: Feature branching web apps
description: Learn how to deploy and manage Azure Web App feature branches in Octopus
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

Octopus is great at managing the progression of your changes through the development, test, and production environments. It also handles branching strategies like hotfixes nicely through the use of channels, allowing you to bypass certain environments and push packages with matching versions rules (like having the word `hotfix` in the version release field) straight to production in an emergency.

But what about feature branches? In this blog post we'll break down exactly what a feature branch is, and how you can manage them in Octopus.

## What is a feature branch?

Before we can model a feature branch in Octopus, we need to understand what a feature branch is to most developers.

[Martin Fowler provides good description of a feature branch](https://martinfowler.com/bliki/FeatureBranch.html):

> A feature branch is a source code branching pattern where a developer opens a branch when she starts working on a new feature. She does all the work on the feature on this branch and integrates the changes with the rest of the team when the feature is done.

I suspect most developers are familiar with working in a feature branch, but it is the finer points, especially related to deployments, contained in the phrase *does all the work on the feature on this branch* that we are interested in.

Specifically, we want to give developers a way to deploy the code they are working on:

* To test it in an environment that can't be easily replicated locally, such as cloud Platform as a Service (PaaS) offerings.
* To easily share the current state of their work with the rest of the team
* Or to provide a stable target for additional testing like end-to-end, performance, or security tests.

Unlike deployments to fixed environments like test or production, feature branches are short lived. They exist while a feature is being developed, but once merged into a mainline branch, the feature branch should be deleted.

Also, a feature branch is not intended to be deployed to production. Unlike a hotfix, which is an emergency production deployment to quickly solve a critical issue, a feature branch is only used for testing.

An potential cost saving consequence of the limited audience a feature branch is exposed to is that you will likely be able to delete the deployments, and their underlying infrastructure, in the evening and redeploy the feature branches again in the morning. Doing so can mean you no longer pay to host applications no one will ever use overnight.

Feature branches are typically processed by a CI system as a convenient way to ensure the tests pass and then produce deployable artifacts. There is a compelling argument to be made that a CI system should produce a deployable artifact (if the code compiles) regardless of the test results given processes like Test Driven Design (TDD) encourage failing tests as a normal part of the design workflow. A question then is how are feature branch artifacts versioned?

Tools like GitVersion [provide examples showing feature branch names included in a SemVer prerelease field](https://gitversion.net/docs/learn/branching-strategies/gitflow/examples), resulting in versions like `1.3.0-myfeature`. Most package management tools have versioning strategies that include components to accommodate a feature branch name. Where a package manager has no versioning guidelines, like Docker repositories, adopting a versioning scheme like SemVer is a good choice.

:::hint
The Maven versioning scheme has a quirk where a version with a qualifier, like `1.0.0-myfeature`, is considered to be a later version that an unqualified version, like `1.0.0`. However, using channel versions rules means qualified versions representing feature branches are not eligible for deployment to production environments, so the ordering of qualified and unqualified versions does not present an issue in Octopus.
:::

With all the above in mind, we can define a feature branch as having the following qualities:

* They are for testing only, and are not exposed in a production environment.
* They are short lived, existing only while a corresponding branch in the version control system exists.
* They may not be needed outside of business hours, providing cost savings if deployments can be shut down overnight.
* They don't get promoted anywhere, and so have a lifecycle that includes a single deployment to a test environment, and eventual clean up.
* They product artifacts versioned in such a way as to identify the source feature branch.

The next step is to model the rules above in Octopus.

## Octopus meta-steps

Octopus includes a limited notion of meta-steps, which is a term we'll use to categorize steps that modify Octopus itself. The **Deploy a release** step is one example, which (as the name suggests) can be used to deploy a release created for an other project. [Script steps can also dynamically generate some Octopus resources](https://octopus.com/docs/infrastructure/deployment-targets/dynamic-infrastructure).

But meta-step functionality is somewhat ad-hoc and limited. What we need is a more comprehensive solution to create and destroy resources within Octopus to reflect the short lived nature of feature branches.

The [Octopus CLI provides some additional functionality](https://octopus.com/docs/octopus-rest-api/octopus-cli), with the ability to create many Octopus resources like releases, channels, and environments. Unfortunately it does not include all the corresponding options to delete these resources, so it is not a complete solution.

Another option is to use the REST API, which exposes every action that the web UI can perform. However, I'd prefer to avoid writing a second CLI tool to manage the complete lifecycle of Octopus resources if possible.

Fortunately the [Octopus Terraform provider](https://registry.terraform.io/providers/OctopusDeployLabs/octopusdeploy/latest/docs) provides exactly what we need. Combining this provider with the existing Terraform deployment steps in Octopus allows us to create our own meta-steps, which in turn means we can manage the ephemeral Octopus resources we need to represent feature branches.

