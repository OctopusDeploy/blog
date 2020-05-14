---
title: Action Containers 
description: Octopus 2020.2 adds the ability to execute deployment actions inside a container 
author: michael.richardson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Product
---

Modern deployments depend on tools. For example: AWS, Azure, and Google command-lines, Terraform, kubectl, Helm, Java, NodeJS, .NET, etcetera, etcetera, _etcetera_.         

Octopus has historically taken an inconsistent approach to these. Some are bundled with the Octopus Server and pushed to deployment targets. Examples of these are Azure CLI, AWS CLI, and Terraform. In other cases, Octopus assumes the dependencies are pre-installed on the targets, e.g. kubectl, Helm, Java. Neither approach is all rainbows.

The bundled dependencies have a number of drawbacks.  They are always out of date, and users often require the latest versions. There is no way for users to pin the versions of them, so if the publishers of the tools introduce breaking changes, we find ourselves unable to update them without potentially breaking users' deployment processes (this is currently a problem with Terraform).   

But by _not_ bundling dependencies, we are pushing our pain onto our users. Spinning up a new machine to use as an Octopus worker is a chore if you have to then install dozens of dependencies. And managing the relationship between various projects' deployment processes and workers is not obvious.  

Fortunately, there is a technology that is perfect for this scenario: _containers_. 

![Containers - The Big Idea by @b0rk](containers-big-idea.jpg)
From: https://twitter.com/b0rk/status/1237464479811633154

