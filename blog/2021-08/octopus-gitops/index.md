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

This post proposes new targets and steps that allow Octopus to deploy to a git repository like any other deployment target, which will position Octopus as the best solution for GitOps left of the git repository:

![](redhatgitops2.png)