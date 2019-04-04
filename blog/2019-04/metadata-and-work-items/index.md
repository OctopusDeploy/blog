---
title: Tracking Your Work From Code to Deployment
description: A look at the new custom metadata capabilities in Octopus.
visibility: public
author: shannon.lewis@octopus.com
published: 2019-04-04
metaImage:
bannerImage:
tags:
 - Jira
 - Work Items
 - Release Notes
---

In this post, we're excited to announce some new features in Octopus Deploy 2019.4 that are focused on tightening the feedback loop in your CI/CD pipeline. These features strengthen the integration between the build servers and Octopus by passing more information about the build down the line.

Octopus 2019.4 includes:

* Build information and work item tracking
* Release notes templates and automatic release notes generation
* Octopus integration with Jira

 We're going to cover [Release Notes Templates](/blog/release-notes-templates) in a separate post, let's have a look at the rest of the features now.

## Build Information and Work Item Tracking

Inherent in building software is the idea that over time the product is the accumulation of the features, issues, and bugs that have been built, released, and deployed.

It follows then that we want to track which features, issues, and bugs have been added into each release of the software, and then which ones are being deployed. We're using the generalized term **work item** to refer to a feature, issue, or bug. To put it another way, Octopus can track work items from code to build to deployment, which provides a deeper understanding of what is included in any given version of the software.

The most common way to communicate changes between releases is with release notes, and the introduction of work items hasn't changed the way Octopus handles release notes on a release itself, but we have added the build information and work items as separate additional information. One change we've made is that deployments now have release notes too, or rather **Release Changes** as we're calling them. These are automatically collected based on the releases in the deployment so you don't need to enter them manually.

"Wait," I hear you say. "Release**s** in the deployment? Don't we deploy **a** release?" Yes, we do, but remember building software is a cumulative process, so what we're deploying is the aggregate of any releases that have occurred since the last deployment to that environment/tenant.

Ok, that was a mouthful, let's have a look at an example.

![Work item accumulation](accumulation.png)

This diagram depicts multiple releases and deployments that have occurred over time, along with which work item details are included for each deployment. In this scenario, each release was immediately deployed to the Dev environment, which results in the simplest case because there was only a single release involved.

The deployments for `1.0.3` illustrate a more complex roll-up of work items. When `1.0.3` was deployed to the Staging environment, it included work items from releases `1.0.2` and `1.0.3`. Similarly, when it was deployed to the Prod environment, it also included the work items from `1.0.1`.

## How Does it Work?

To make this work we have updated our build server plugins to include a new `Octopus Metadata` step. This step gathers metadata about the build, including parsing the commit messages looking for work item references and includes all of this in a call to Octopus. It's similar to pushing a package and requires the packageID and version of the package that the metadata describes.

The package itself doesn't have to be one you are pushing to Octopus. This is an important part, Octopus is receiving the metadata for a given packageID and version, and it stores that for use when creating releases and deployments. The package itself can be of **any format and from any feed**, including container images from a container repository.

The metadata includes links to the build that created the package, as well as any work items included in the package.

For packages that do get pushed to Octopus, you will see the metadata when you view them in the library.

![Package details metadata](package-detail.png)

This metadata also appears on the release, deployment preview, and task pages. For packages from external feeds, the release is the first point at which you will see the package's metadata in the Octopus portal.

![Release metadata](release-work-items.png)

## Deployment Variables

As we mentioned above, the deployments have been extended to include "Release Changes." An important point about this is that the **deployments will always aggregate release notes from the release(s)** into the Release Changes, even if there is no metadata and work items.

We're going to talk about this more in [another post](/blog/release-notes-templates).

## Deploy a Release Step

When is a package not a package? When it's a child project in a _Deploy a Release_ step. That scenario is covered too. The metadata and work items will not only be calculated on the releases/deployments in the child projects, but they also get aggregated into the releases/deployments on the "parent" project.

## Build Servers

In the initial release, our Bamboo and TeamCity plugins have been updated to include the new Metadata step.

We are still working on the updates for the Azure DevOps plugin, which should be available soon.

Jenkins support will also be coming, but we don't have a timeframe for that right now.

## Jira

So far everything we've talked about is build server and Octopus centric and holds true for anyone using Octopus. Let's look at what else you get if you're using hosted Jira.

The new Jira Issue Tracker extension in Octopus ties in with the [deployment dashboard](https://confluence.atlassian.com/bamboo/viewing-bamboo-activity-in-jira-applications-399377384.html) functionality in Jira, so you can get deployment feedback the same as if you were using BitBucket Pipelines.

As you do the deployments in Octopus, it feeds information back to Jira in real-time, and you will see the deployments appear in Jira.

![Jira Deployments](jira-deployment.png)

A subset of the Jira integration features are also available for those with an on-premises instance, see our [documentation](https://g.octopushq.com/JiraIssueTracker) for more details.

## Wrap up

Tighter integration between the major parts in the CI/CD pipeline helps streamline the flow of information. These new integration features with Jira are a great example of this, so if you're using Jira and Octopus now's the time to integrate!
