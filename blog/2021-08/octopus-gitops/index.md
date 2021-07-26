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

[We're also seeing GitOps being mentioned in customer calls](https://octopusdeploy.slack.com/archives/C0223UFAEAY/p1627262125186800):

> "At Crowe, Octopus is our first choice." Reasons to not go with OD: Kubernetes (they prefer a more git-ops model with Flux)"
> I've heard this on a few calls re: k8s. Flux and Argo are usually mentioned

While GitOps is a nebulous term, the [GitOps Working Group](https://github.com/gitops-working-group/gitops-working-group) has provided some initial principals:

* Declarative Configuration: All resources managed through a GitOps process must be completely expressed declaratively.
* Version controlled, immutable storage: Declarative descriptions are stored in a repository that supports immutability, versioning and version history. For example, git.
* Automated delivery: Delivery of the declarative descriptions, from the repository to runtime environment, is fully automated.
* Software Agents: Reconcilers maintain system state and apply the resources described in the declarative configuration.
* Closed loop: Actions are performed on divergence between the version controlled declarative configuration and the actual state of the target system.

Today the emphasis on GitOps is what happens after configuration has been committed to a git repository (which I'll refer to as "right of the git repo"). Tools like [Argo CD](https://argoproj.github.io/argo-cd/) and [Flux](https://fluxcd.io/) are used to manage Kubernetes clusters, while there are some early homegrown solutions appearing for other declarative tools like Terraform and Ansible.

This diagram from the post [Ops by pull request: an Ansible GitOps story](https://www.ansible.com/blog/ops-by-pull-request-an-ansible-gitops-story) provides a nice overview of the state of GitOps, although I suspect not in the way the original authors intended:

![](redhatgitops.png)

What we see time and again in the GitOps space is developers and operations staff committing directly to a git repository, and then using the "magic of GitOps" to realise its many benefits.

We know from our experience deploying applications that there is a huge amount of work required to scale up deployments. You need environments, tenants, dashboards, interventions, templates, security, automated testing, reporting and more to manage deployments at scale. I propose paradigms like GitOps also need these things, and anyone implementing GitOps at scale today is most likely twisting a CI server into knots trying to implement these features.

This post proposes new targets and steps that allow Octopus to deploy to a git repository like any other deployment target, which will position Octopus as the best solution for GitOps "left of the git repo":

![](redhatgitops2.png)

## What problems are we trying to solve?

[To quote a recent panel discussion on GitOps](https://youtu.be/GrHmeAxfEkM?list=PL2KXbZ9-EY9TRND2YHxordGt8pOw5r45R&t=1177):

> The interface to operations is now through git

GitOps views the evolution of operations and deployments as:

1. Starting with manual configuration performed through SSH or RDP.
2. Processes automated through CLI tools like `kubectl`.
3. Finally git commits by end users (or tooling like Octopus) defining the declarative state of a system to be reified by an agent of some kind.

GitOps identified running adhoc commands with tools like `kubectl` as a weakness in a delivery pipeline operating at scale. Octopus identifies running adhoc `git` commits as a similar weakness. Because any sufficiently advanced GitOps workflow must eventually treat a git repo like a database.

One of the stated goals of team steps is to prefer [declarative over imperative](https://github.com/OctopusDeploy/Architecture/blob/main/Steps/StepDesignGuidelines.md#declarative-over-imperative), meaning new steps will default to creating declarative representations of deployments rather than execute custom commands against an SDK. This fits nicely with the GitOps paradigm.

The three problems this RFC aims to solve are deploying declarative state to a git repository, integrating common git operations like pull requests into existing deployment processes or runbooks, and allowing GitOps to scale.

### Deploying declarative state to git

Steps like `Deploy Kubernetes containers` and the upcoming ECS deployment have been designed from the ground up to build declarative templates. By committing these templates to a git repository rather than applying them directly, Octopus can replace the role of a developer or operations staff performing manual updates.

### Managing pull requests

Pull requests are a very important aspect of GitOps, as this is how changes are reviewed and authorized. Octopus will provide the opportunity to review, comment on, approve, and merge PRs as part of a standard deployment process or runbook.

### Allowing GitOps to scale

GitOps is all about scale. To implement the infrastructure required to support GitOps, teams must reach an advanced point in their deployment and DevOps maturity. Most GitOps workflows focus heavily on responding to changes to a git repo, bit don't offer a good solution for managing the incoming changes.

GitOps at scale means many teams committing to many repositories with many PRs to be processed. This in turn means providing notifications or a dashboard of pending reviews, which Octopus can provide.

Fine grained security with git repos inevitably requires many (probably dozens, possibly hundreds) of individual repos as the git security model can typically only scope permissions to a branch or a repo. Octopus can provide a source of truth for these many repos, while optionally being the single system that can write to them.

Reporting is a challenge in GitOps. To report on commits to a git repo, a reporting tool must be able to query all the commits and infer meaning from the changes. Git will never scale to the point where it can be read like a structured database, so this won't be a viable solution. This means today the tooling "to the right of the repo" must then interpret the changes being applied in the context of reportable metrics. This is possible, but will always be limited by the ability of tooling to infer intention (like "a new version of the web application was deployed") from a change to a file (like "the file webapplication.yaml had the image field updated"). 

Octopus offers a solution here because a deployment process or runbook captures the intention of an action, which can then be used to generate meaningful metrics.

The only solution provided by GitOps to ensure consistent and well formed resources are committed to a repo are pull requests. This does not scale well, as relying on manual reviews of complex YAML or JSON to ensure rules like labels are consistently applied or memory limits are within a certain range will eventually overwhelm reviewers. Octopus provides a solution by committing well known resources defined in repeatable steps.

