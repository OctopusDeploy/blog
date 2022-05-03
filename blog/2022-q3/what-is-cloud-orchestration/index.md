---
title: What is cloud orchestration?
description: A brief summary of the post, 170 characters max including spaces.
author: terence.wong@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage:
bannerImage:
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags:
  - DevOps
  - Runbooks Series
  - Runbooks
---

<!-- see https://github.com/OctopusDeploy/blog/blob/master/tags.txt for a comprehensive list of tags -->
## Introduction

Cloud solutions have gained popularity over traditional IT systems. A [study by Gartner](https://www.gartner.com/en/newsroom/press-releases/2022-02-09-gartner-says-more-than-half-of-enterprise-it-spending) indicates that revenue for cloud products are set to overtake traditional IT solutions in 2025.

![Gartner Cloud Adoption](gartner-cloud-adoption.png "width=500")

To achieve the revenue gains, there is a need to optimize cloud solutions to make them more efficient and cost effective. Cloud orchestration is the co-ordination and automation of workloads, resources and infrastructure on public and private cloud environments. Cloud orchestration is the automation of the whole cloud system. Each part in the system should work together to produce an efficient, well-oiled machine. Cloud automation is a subset of cloud orchestration that is focused on automating the individual parts of a cloud system. Both cloud orchestration and automation complement each other to produce an automated cloud system.

Cloud services are delivered in three main models:

### Software as a Service (SaaS)

SaaS is a software licensing and delivery model where a software solution is provided on-demand and hosted by the provider. SaaS solutions often have a subscription fee or freemium pricing model to access the service. The benefits of this approach is that users do not have to install and host the application and can access only what they need. Some examples of SaaS solutions are Dropbox, Google Apps like Gmail or Netflix.


### Platform as a Service (PaaS)

PaaS platforms give developers complete cloud development and deployment environment. Developers can load operating systems and development tools on VMs. PaaS provides a contained environment to build cloud applications without needing to manage licensing or underlying application infrastructure. Platforms like Microsoft Azure, Google Cloud Platform or Amazon Web Services are examples of PaaS.


### Infrastructure as a Service (IaaS)

IaaS provides an on-demand service to deploy IT infrastructure such as virtual machines, servers, networks, and storage. IaaS is structured as pay-as-you-go where users pay for the infrastructure they need, when they need it. Examples of IaaS platforms include Digital Ocean, or AWS EC2.

SaaS systems are often built on IaaS and PaaS platforms and PaaS platforms are often built on IaaS platforms. The diagram below shows how SaaS, IaaS and PaaS work together to deliver cloud systems:

<!-- Good image https://azure.microsoft.com/en-au/overview/what-is-paas/ -->

## Why it's important


## Benefits



Efficiency
Cost Reductions
Supports DevOps
Security

## Different tooling

Here are some cloud orchestration tools:

### Terraform

Terraform is an open source infrastructure as code tool. You can specify your infrastructure in configuration files which are interpreted to deploy infrastructure on the cloud. This tool is used by many cloud providers as a common framework to deploy infrastructure solutions.

### Kubernetes

### Platform as a Service Cloud Automation tools

Many PaaS cloud providers have tools that allow cloud automation. These tools include:

- AWS Cloud Formation
- Microsoft Azure Automation
- IBM Cloud Orchestrator
- Google Cloud Composer

These tools allow developers to automate their cloud environments through infrastructure as code, deployment management GUIs, and integrations to other cloud solutions within the PaaS system.

### RedHat Ansible

Cloudify
Morpheus

## How Octopus fits in

## Conclusion
