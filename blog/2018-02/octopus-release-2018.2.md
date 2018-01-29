---
title: "Octopus February Release 2018.2"
description: What's new in Octopus 2018.2
author: matthew.casperson@octopus.com
visibility: private
metaImage: metaimage-shipping-2018-2.png
bannerImage: blogimage-shipping-2018-2.png
published: 2018-01-24
tags:
 - New Releases
---

Octopus 2018.2 brings a number of exciting new features including the [much requested step to deploy a release](https://octopusdeploy.uservoice.com/forums/170787-general/suggestions/9811932-allow-project-dependencies-so-deploying-one-proj), the ability to deploy AWS CloudFormation templates, delete existing CloudFormation stacks, and run scripts with the AWS CLI.

## In this post

!toc

## AWS Support

This release introduces 3 new steps related to AWS.

The first allows custom scripts to be run against the AWS CLI. Octopus provides the AWS credentials and the AWS CLI itself, making it easy to interact with AWS resources as part of a deployment.

The two other steps allow you to deploy CloudFormation templates and delete existing CloudFormation stacks. Octopus takes care of the parameters and outputs, and allows you to deploy CloudFormation templates entered directly in the step or from an external package.

## Michael Richardson's new steps

## Rob Erez's package changes

## Improvements to audit logs

When resources referenced by other resources are deleted we will now log how and why they changed, a key one being changes to Project Variables. There's also some other resource types are now being audited now and/or with more detail.

## Breaking Changes

TODO - Are there breaking changes?

## Upgrading

As usual [steps for upgrading Octopus Deploy](https://octopus.com/docs/administration/upgrading) apply. Please see the [release notes](https://octopus.com/downloads/compare?to=2018.2.0) for further information.

## Wrap up

That’s it for this month. Feel free leave us a comment and let us know what you think! Go forth and deploy!
