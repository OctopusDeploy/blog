---
title: Tracking your work from code to deployment
description: A look at the new custom metadata capabilities in Octopus.
author: shannon.lewis@octopus.com
visibility: private
published: 2019-03-28
metaImage: 
bannerImage: 
tags:
 - Jira
 - Work Items
---

In this post we're excited to announce some new features in Octopus that are focused on tightening the feedback loop in your CI/CD pipeline. These features strengthen the integration between the build servers and Octopus by passing more information about the build down the line.

## Build Information and Work-Items

Inherent in building software is the idea that over time the product is the accumulation of the features/issues/bugs that have been built/released/deployed.

It follows then that we commonly want to track which features/issues/bugs are being built into each release of the software, and then which ones are being deployed. For our purposes, we're using the generalized term **work-item** to refer to a feature, issue or bug. To put it another way, Octopus will now be able to track work-items from code to build to deployment.

The value of the work-items is in understanding what is included in any given version of the software. The common way to communicate this is using Release Notes. 

The introduction of work-items hasn't changed the way Octopus handles release notes on a release itself, but we've added the build information and work-items as separate information. What we've changed is that deployments now have release notes too, or rather **ReleaseChanges** as we're calling them. These aren't something you need to manually enter, they are automatically collected based on the releases in the deployment. 

"Wait", I hear you say. "Release**s** in the deployment?, don't we deploy **a** release?". Yes we do, but remember building software is a cumulative process, so what we're deploying is the aggregate of any releases that have occurred since the last deployment to that environment/tenant.

Ok that was a mouthful, let's have a look at an example.

![Work-item accumulation](accumulation.png)

This diagram depicts a number of releases and deployments that have occurred over time, along with which work-item details are included for each deployment. In this scenario each release was immediately deployed to the Dev environment, which results in the simplest case because there was only a single release involved.

The deployments for `1.0.3` illustrate a more complex roll-up of work-items. When `1.0.3` was deployed to the Staging environment, it included work-items from releases `1.0.2` and `1.0.3`. Similarly, when it was deployed to the Prod environment it also included the work-items from `1.0.1`.

## How does it work?

To make this work we have updated our build server plugins to include a new `Octopus Metadata` step. This step gather metadata about the build, including parsing the commit messages looking for work-item references, and includes all of this in a call to Octopus. It's similar to pushing a package, and requires the packageId and version of the package that the metadata relates to.

The package itself doesn't have to be one you are pushing to Octopus. This is an important part, Octopus is receiving the metadata for a given packageId and version, and it stores that away for use when creating releases and deployments. The package itself can be of **any format and be coming from any feed**, including being a container image in a container repository.

The metadata includes links to the build that created the package, as well as any work-items included in the package. 

For packages that do get pushed to Octopus, you will see the metadata when you view them in the library.

![Package details metadata](package-detail.png)

This metadata then also appears on the Release, Deployment Preview, and Task pages. For packages from external feeds the Release is the first point at which you will actually see the package's metadata in the Octopus portal.

![Release metadata](release-work-items.png)

## Release Changes and Deployment Variables

As we mentioned above, the deployments have been extended to include "Release Changes". An important point about this is that the **deployments will always aggregate release notes from the release(s)** into the Release Changes, even if there is no metadata and work items.

For each release related to the deployment, the Release Changes includes a version (the release version), the release notes, and a list of work-items.

We wanted to call this out explicitly, because the new variable data we provide for deployments is a little different to what we provide for release notes on the release.

On a release, you can enter release notes and we accept them in markdown format. This works fine if you know how you are wanting to consume them, you actually use markdown syntax if it suits and plain text if it doesn't. A common use of this information is on our Email step, and for it you want HTML not straight Markdown.

So we started down the path of generating a formatted notes field on the deployment and thought, "Wait, this will need to be consumed in different ways in different scenarios. If we make it Markdown that won't work for anyone trying to use it for emails, if we make it HTML that won't work for other scenarios". So what we settled on was providing the raw JSON data in the variable and then you have complete control over how you want to format it. Our documentation **TODO!** has a sample of how to consume the data in the email step to format a message however you want it to look.

## Deploy a release step

When's a package not a package? When it's a child project in a _Deploy a Release_ step. That scenario is covered too. The metadata and work-items will not only be calculated on the releases/deployments in the child projects, they get aggregated into the releases/deployments on the "parent" project.

## Build servers

In the initial release our Bamboo and TeamCity plugins have been updated to include the new Metadata step. 

TODO: the following may or may not be true by the time we publish!!!!

We are still looking at/working on how this integration fits in with Azure DevOps.

Jenkins support will be coming, but we don't have a timeframe on that at this point.

## Jira

So far everything we've talked about is actually build server and Octopus centric, and holds for anyone using Octopus. Let's have a look now at what you get extra if you're using hosted Jira.

The new Jira Issue Tracker extension in Octopus ties in with the [deployment dashboard](https://confluence.atlassian.com/bamboo/viewing-bamboo-activity-in-jira-applications-399377384.html) functionality in Jira, so you can get deployment feedback the same as you would if you were using BitBucket Pipelines.

As you do the deployments in Octopus, it feeds information back to Jira in real-time and you will see the deployments appear in Jira.

![Jira Deployments](jira-deployment.png)

For more information on how to configure the integration with Jira, see our documentation. TODO Url

## Wrap up

And that's it! Well for now anyway. As the CI/CD world continues to mature and evolve we're expecting to see more and more examples of this richer integration and feedback throughout the pipeline, so watch this space.

As always, please leave us feedback below and happy deployments!