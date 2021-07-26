---
title: Octopus and GitOps
description: Learn how Octopus is evolving to be the best deployment tool in 2022.
author: matthew.casperson@octopus.com
visibility: private # DO NOT CHANGE THIS!!!! This is not a public post!
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

GitOps is a relative newcomer to the deployment scene, with the term having been coined by WeaveWorks in a blog post called [Operations by Pull Request](https://www.weave.works/blog/gitops-operations-by-pull-request) in 2017.

[InfoQ reports that GitOps is now being adopted by an early majority](https://www.infoq.com/articles/devops-and-cloud-trends-2021):

> Both GitOps and site reliability engineering (SRE) practices are increasingly being adopted.

![](infoq.jpg)

*InfoQ DevOps and Cloud InfoQ Trends Report - July 2021*

While GitOps is a nebulous term, the [GitOps Working Group](https://github.com/gitops-working-group/gitops-working-group) has provided some initial principals:

* Declarative Configuration: All resources managed through a GitOps process must be completely expressed declaratively.
* Version controlled, immutable storage: Declarative descriptions are stored in a repository that supports immutability, versioning and version history. For example, git.
* Automated delivery: Delivery of the declarative descriptions, from the repository to runtime environment, is fully automated.
* Software Agents: Reconcilers maintain system state and apply the resources described in the declarative configuration.
* Closed loop: Actions are performed on divergence between the version controlled declarative configuration and the actual state of the target system.

Today the emphasis on GitOps is what happens after configuration has been committed to a git repository. Tools like [Argo CD](https://argoproj.github.io/argo-cd/) and [Flux](https://fluxcd.io/) are used to manage Kubernetes clusters, while there are some early homegrown solutions appearing for other declarative tools like Terraform and Ansible.

This diagram from the post [Ops by pull request: an Ansible GitOps story](https://www.ansible.com/blog/ops-by-pull-request-an-ansible-gitops-story) provides a nice overview of the state of GitOps, although I suspect not in the way the original authors intended:

![](redhatgitops.png)

What we see time and again in the GitOps space is developers and operations staff committing directly to a git repository, and then using the "magic of GitOps" to realise its many benefits.

We know from our experience deploying applications that there is a huge amount of work required to scale up deployments. You need environments, tenants, dashboards, interventions, templates, security, automated testing, reporting and more to manage deployments at scale. I propose paradigms like GitOps also need these things, and anyone implementing GitOps at scale today is most likely twisting a CI server into knots trying to implement these features.

This post proposes new targets and steps that allow Octopus to deploy to a git repository like any other deployment target, which will position Octopus as the best solution for GitOps "left of the git repository":

![](redhatgitops2.png)

## What problems are we trying to solve?

[To quote a recent panel discussion on GitOps](https://youtu.be/GrHmeAxfEkM?list=PL2KXbZ9-EY9TRND2YHxordGt8pOw5r45R&t=1177):

> The interface to operations is now through git

GitOps views the evolution of operations and deployments as starting with manual configuration performed through SSH or RDP, to processes automated through CLI tools like `kubectl`, and finally to commits to a git repository by end users (or tooling like Octopus) defining the declarative state of a system to be reified by an agent of some kind.

One of the stated goals of team steps is to prefer [declarative over imperative](https://github.com/OctopusDeploy/Architecture/blob/main/Steps/StepDesignGuidelines.md#declarative-over-imperative), meaning new steps will attempt to create declarative representations of deployments rather than execute custom commands against an SDK. This fits nicely with the GitOps paradigm.

The two problems this RFC aims to solve are deploying declarative state to a git repository, integrating common git operations like pull requests into existing deployment processes or runbooks, and allowing GitOps to scale.

### Deploying declarative state to git

Steps like `Deploy Kubernetes containers` and the upcoming ECS deployment have been designed from the ground up to build declarative templates. By committing these templates to a git repository rather than applying them directly, Octopus can replace the role of a developer or operations staff performing manual updates.

### Managing pull requests

Pull requests are a very important aspect of GitOps, as this is how changes are reviewed and authorized. Octopus will provide the opportunity to review, comment on, approve, and merge PRs as part of a standard deployment process or runbook.

### Allowing GitOps to scale

GitOps is all about scale. To implement the infrastructure required to support GitOps requires teams to reach an advanced point in their deployment and DevOps maturity. GitOps at scale means many teams committing to many repositories with many PRs to be processed. It also means integrating tools like CI servers to allow programatic changes, such as replacing a Docker tag in a declarative template when a new Docker image is built and published.

I propose that any sufficiently advanced GitOps workflow must treat a git repository like a structured data store because:

* Automated tooling must know the exact field in the exact file to update when deploying new images.
* To progress changes across environments, each git repository hosting an environment must be substantially similar to the other environments.
* Reporting tools must be able to inspect the state of a git repository and reason about the changes (assuming reporting is done "right of the git repo").
* Resources themselves will hold structured data, for example Kubernetes configmaps and secrets, that must map to outside values in a predictable manner.
* Depending on how granular git repositories are, rollbacks may require resetting the state of a well known file rather than reverting to a previous commit, so individual files have tight colorations to individual "deployments".
* PRs only make sense when small changes are made to similar resources.
* Any team advanced enough to implement GitOps must be deploying well known resources with common constraints.
* Nobody goes to the trouble of implementing GitOps just so they can make YOLO commits of whatever resources take their fancy.