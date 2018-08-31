---
title: Octopus Release 2018.8
description: Octopus 2018.8 - Script Step Packages and Kubernetes Alpha 
author: michael.richardson@octopus.com
visibility: private
published: 2018-09-03
tags:
 - New Releases
---

## In This Post

!toc

## Release Tour

_To Be Added_

## Script Step Packages++

In 2018.8 the family of script steps (_Run a Script_, _Run an Azure PowerShell Script_, _Run an AWS CLI Script_) gain some new super-powers.   

We are making consuming packages from script steps much easier and more powerful.  Multiple package references can now be added, and each package can be optionally extracted.  Container images can also be referenced.  

We have an entire [post on these enhancements](https://octopus.com/blog/script-step-packages), so please have a read.

## Kubernetes Alpha

We are very excited to be including the first-draft of Kubernetes support in Octopus!  This functionality is being released behind a feature-flag, and is designed to enable us to receive feedback from some real users. Although we don't currently recommend using this for production work-loads, we would encourage anyone interested to take a peek and all feedback is very welcome. 

![Kubernetes Feature Flag](k8s-feature-flag.png "width=500")

![Kubernetes Step](kubernetes-steps.png "width=500")

## AWS ECR Feed

A new `AWS Elastic Container Registry` feed type has been added. This makes configuring AWS credentials to access container images from ECR much more convenient. 

![ECR Feed](ecr-feed.png "width=500")

## Upgrading

As usual [steps for upgrading Octopus Deploy](https://octopus.com/docs/administration/upgrading) apply. Please see the [release notes](https://octopus.com/downloads/compare?to=2018.8.0) for further information.