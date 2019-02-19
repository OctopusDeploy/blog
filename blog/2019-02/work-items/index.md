---
title: Tracking your work from code to deployment
description: A look at the new integration between Octopus and Jira.
author: shannon.lewis@octopus.com
visibility: private
published: 2019-02-28
metaImage: 
bannerImage: 
tags:
 - Jira
 - Work Items
---

In this post we're excited to announce some new features in Octopus that are focused on enhancing the feedback loop in your CI/CD pipeline. These features provide feedback on release and deployment progress and dovetail into the new release/deployment functionality Atlassian are introducing in Jira.

This gives those creating issues/bugs etc in Jira more timely information on how the implementation is progressing. It also gives those doing the implementation more timely information on where their code is at in the deployment pipeline.

Let's have a closer look at what's now in the box.

## Work Items

Inherent in building software is the idea that over time the product is the accumulation of the features/issues/bugs that have been built/released/deployed.

It follows then that we commonly want to track which features/issues/bugs are being built into each release of the software, and then which ones are being deployed.

For the new functionality we are introducing, we've used the generalized term _work item_ to refer to a feature, issue or bug. So what we're saying is, Octopus will now be able to track work items from build to deployment.

The value of the work items for the consumer is understanding what is included in any given version of the software. The common way to communicate this is using Release Notes. 

The introduction of work items hasn't changed the way Octopus handles release notes on a release itself, we've added the work items as separate information. What we've conceptually changed though is that deployments now have release notes too. These aren't something you can manually enter, they are automatically aggregated based on the releases in the deployment. 

Wait, "release**s** in the deployment", don't we deploy **a** release? Yes we do, but remember building software is a cumulative process, so what we're deploying to an environment/tenant is the aggregate of any releases that have occurred since the last deployment to the environment/tenant.

Ok that was a mouthful, let's have a look at an example.

![Work item accumulation](accumulation.png)

This diagram depicts a number of releases and deployments that have occurred over time, along with which work item details accumulated for each deployment. In this scenario each release was immediately deployed to the Dev environment, which results in the simplest accumulation because there was only a single release involved.

The deployments for `1.0.3` illustrate a more complex accumulation of work items. When `1.0.3` was deployed to the Staging environment, it accumulated work items from releases `1.0.2` and `1.0.3`. Similarly, when it was deployed to the Prod environment the accumulation also included the work items from `1.0.1`.



## Deploy a release step

TODO: work out if we're going to support this in the initial release. A couple of customers have asked about support for this, so we should call it out either way.