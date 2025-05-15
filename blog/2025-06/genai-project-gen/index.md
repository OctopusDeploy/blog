---
title: Generating projects with GenAI
description: Learn about how we're using GenAI to generate projects in Octopus Deploy.
author: matthew.casperson@octopus.com
visibility: provate
published: 2025-06-01-1400
metaImage: img-blog-laptop-cogs-cloud.png
bannerImage: img-blog-laptop-cogs-cloud.png
bannerImageAlt: Stylized laptop screen showing Octopus logo connected to cogs in the cloud, with a clipboard to the right.
tags:
  - Product
isFeatured: false
---

## Introduction

The DevOps landscape is growing increasingly complex. Cloud providers regularly add new platforms, the Kubernetes ecosystem is undergoing a Cambrian explosion, and all of this while DevOps teams are expected to address security, observability, scale, and compliance in every step of their processes.

Octopus has an amazing ability to model the complex environments found in modern enterprises, but configuring the environments, feeds, accounts, lifecycles, projects, runbooks, and targets that make up any real world deployment scenario can be a challenge, let alone doing so according to best practises with patterns that will scale.

To help teams get up and running quickly on Octopus, we're introducing a new feature that populates an Octopus space with common projects based on our opinions on best practices based on a prompt like `Create a Kubernetes project called "Web App"`. This functionality is powered by GenAI, and in this post I'll describe how a simple prompt becomes a fully functional project in your Octopus instance.

## Building the templates

GenAI is only as good as the data it has learned from. For GenAI to build a project, along with all the supporting resources, we start by hand crafting a number of template projects according to a set of best practices that we've evolved over the years. At this stage the projects are nothing more than regular projects manually created via the web UI.

Best practices capture aspects like variable and target tag naming conventions, release versioning schemes, worker images, supporting runbooks, and much more. There are hundreds of configurable fields in an Octopus project, and we have a continually evolving set of opinions to provide DevOps teams with a solid foundation for their deployment workflows.

Importantly, we do not train an LLM on any customer data.

Once we have a template project that embodies our best practices, the next step is to serialize the project configuration in a format that can be passed to an LLM.

## Serializing the template

## Training the LLM

## Generating the project

## Security considerations

## Refining the instructions

## Conclusion