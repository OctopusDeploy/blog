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

Cloud solutions have gained popularity over traditional IT systems. A [study by Gartner](https://www.gartner.com/en/newsroom/press-releases/2022-02-09-gartner-says-more-than-half-of-enterprise-it-spending) indicates that revenue for cloud products are set to overtake traditional IT solutions in 2025. To achieve the revenue gains, there is a need to optimize cloud solutions to make them more efficient and cost effective. Cloud orchestration is the co-ordination and automation of workloads, resources and infrastructure on public and private cloud environments. Cloud orchestration is the automation of the whole cloud system. Each part in the system should work together to produce an efficient, well-oiled machine. Cloud automation is a subset of cloud orchestration that is focused on automating the individual parts of a cloud system. Both cloud orchestration and automation complement each other to produce an automated cloud system.

![Gartner Cloud Adoption](gartner-cloud-adoption.png "width=500")

## As a Service Models

Cloud services are delivered in three main models:

- Software as a Service (SaaS)
- Platform as a Service (PaaS)
- Infrastructure as a Service (IaaS)

SaaS is a software licensing and delivery model where a software solution is provided on-demand and hosted by the provider. SaaS solutions often have a subscription fee or freemium pricing model to access the service. The benefits of this approach is that users do not have to install and host the application and can access only what they need. Some examples of SaaS solutions are Dropbox, Google Apps like Gmail or Netflix. PaaS platforms give developers complete cloud development and deployment environment. Developers can load operating systems and development tools on VMs. PaaS provides a contained environment to build cloud applications without needing to manage licensing or underlying application infrastructure. Platforms like Microsoft Azure, Google Cloud Platform or Amazon Web Services are examples of PaaS. IaaS provides an on-demand service to deploy IT infrastructure such as virtual machines, servers, networks, and storage. IaaS is structured as pay-as-you-go where users pay for the infrastructure they need, when they need it. Examples of IaaS platforms include Digital Ocean, or AWS EC2. SaaS systems are often built on IaaS and PaaS platforms and PaaS platforms are often built on IaaS platforms. Together, as a service systems allow developers to acheive cloud orchestration and automation.  The diagram below shows how SaaS, IaaS and PaaS work together to deliver cloud solutions:

<!-- Good image https://azure.microsoft.com/en-au/overview/what-is-paas/ -->

## Benefits of Cloud Orchestration

Cloud orchestration and automation allows developers to automate every part of their cloud solution. Orchestration and automation leads to:

- Increased efficiency
- Cost reductions
- Support for DevOps
- Security

When processes are automated in a cloud solution, the cloud solution can detect when peak times occur and deploy extra services to prevent services from being overloaded. Cloud solutions can also shut down any idle processes that are not needed. The optimized allocation of resources leads to increased efficiency of the platform and reduced costs. Cloud orchestration supports the DevOps framework by allowing continuous integration, monitoring and testing. All services are managed which leads to frequent updates and faster troubleshooting. Faster troubleshooting leads to improved security as vulnerabilities can be patched quickly.

## Cloud Orchestration tools

Terraform is an open source infrastructure as code tool. You can specify your infrastructure in configuration files which are interpreted to deploy infrastructure on the cloud. The benefit of infrastructure as code is that infrastructure configurations can be saved and restored between versions. Terraform is used by many cloud providers as a common framework to deploy infrastructure solutions. Kubernetes is a container orchestration tool developed by Google. Containers are lightweight computing units that make up a larger application. Kubernetes works with cloud providers to manage and deploy containers on infrastructure. Resources can be scaled up or down depending on demand which saves costs and increases the reliability of the application.

Many PaaS cloud providers have tools that allow cloud automation, such as:

- AWS Cloud Formation
- Microsoft Azure Automation
- IBM Cloud Orchestrator
- Google Cloud Composer

These tools allow developers to automate their cloud environments through infrastructure as code, deployment management GUIs, and integrations to other cloud solutions within the PaaS system. There are dedicated cloud orchestration tools such as:

- RedHat Ansible
- Cloudify
- Morpheus

These dedicated tools provide features such as cloud provisioning, configuration management, and automation. All cloud orchestration tools are designed to work with the major technologies such as Terraform and Kubernetes. The choice of tool will depend on IT budget, preferred languages, the location of preexisting deployments or other application specific requirements.

## How Octopus fits in

## Conclusion
