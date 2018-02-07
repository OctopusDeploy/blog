---
title: "Octopus February Release 2018.2"
description: What's new in Octopus 2018.2
author: matthew.casperson@octopus.com
visibility: private
metaImage: metaimage-shipping-2018-2.png
bannerImage: blogimage-shipping-2018-2.png
published: 2018-02-08
tags:
 - New Releases
---

Octopus 2018.2 brings a number of exciting new features including the [much requested step to deploy a release](https://octopusdeploy.uservoice.com/forums/170787-general/suggestions/9811932-allow-project-dependencies-so-deploying-one-proj), the ability to deploy AWS CloudFormation templates, delete existing CloudFormation stacks, and run scripts with the AWS CLI.

## In this post

!toc

## AWS Support

This release introduces 3 new steps related to AWS.

![AWS Steps](aws-steps.png "width=500")

The first allows custom scripts to be run against the AWS CLI. Octopus provides the AWS credentials and the AWS CLI itself, making it easy to interact with AWS resources as part of a deployment.

The two other steps allow you to deploy CloudFormation templates and delete existing CloudFormation stacks. Octopus takes care of the parameters and outputs, and allows you to deploy CloudFormation templates entered directly in the step or from an external package.

## Coordinating Projects with the Deploy a Release Step

![Deploy Release Step Card](deploy-release-step/deploy-release-card.png)   

One of our most popular [UserVoice suggestions](https://octopusdeploy.uservoice.com/forums/170787-general/suggestions/9811932-allow-project-dependencies-so-deploying-one-proj) for a while now has been the ability to coordinate multiple Octopus projects, by having one trigger the deployment of another.

With this release we are proud to introduce the [Deploy a Release step](deploy-release-step/deploy-release-step.md).  

We hope this will enable many powerful multi-project scenarios.

![Example Deploy Release Step Project Process](deploy-release-step/voltron-project-process.png "width=500")

## Improvements for large dashboards

We'd also like to highlight one small but significant change that is a great addition for large Octopus instances. If you had a large number of environments or tenants, your dashboards that were very hard to read and often you'd be need to scroll horiztontally to see more content. The good news is that we've updated the dashboards with a fixed first column and headers so they so they far easier to read and work with. 

![Example large dashboard with scrolling](busy-dashboards.gif)

## Improvements to audit logs

When resources referenced by other resources are deleted we will now log how and why they changed, a key one being changes to Project Variables. There's also some other resource types are now being audited now and/or with more detail.

## Breaking Changes
### Improving our Packages API
In anticipation of some upcoming new feed types we have hit the point where we were forced to revisit how we expose the packages API for external feeds, and how we store cached packages for deployments.
Unless you are hitting the Octopus API directly to search through your external feeds or rely on specific naming of the cached packages, then there should be almost no impact to you. One side effect of the change to package cache names is that the current packages cache on the server and tentacles will be no longer checked so new deployments will use the new package names.
More details about these changes are available in the GitHub tickets ["Packages API does not meet the requirements of our expanding feed types #4114"](https://github.com/OctopusDeploy/Issues/issues/4114) and ["Modify the cache naming format to allow for new feed formats #4211"](https://github.com/OctopusDeploy/Issues/issues/4211).


## Upgrading

As usual [steps for upgrading Octopus Deploy](https://octopus.com/docs/administration/upgrading) apply. Please see the [release notes](https://octopus.com/downloads/compare?to=2018.2.0) for further information.

## Wrap up

That’s it for this month. Feel free leave us a comment and let us know what you think! Go forth and deploy!
