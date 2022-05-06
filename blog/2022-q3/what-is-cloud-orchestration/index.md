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

Cloud solutions have gained popularity over traditional IT systems. A [study by Gartner](https://www.gartner.com/en/newsroom/press-releases/2022-02-09-gartner-says-more-than-half-of-enterprise-it-spending) indicates that revenue for cloud products will overtake traditional IT solutions in 2025. There is a need to optimize cloud solutions to make them more efficient and cost-effective to achieve revenue gains. Cloud orchestration is the coordination and automation of workloads, resources, and infrastructure in public and private cloud environments. Cloud orchestration is the automation of the whole cloud system. Each part of the system should work together to produce an efficient system. Cloud automation is a subset of cloud orchestration focused on automating the individual components of a cloud system. Cloud orchestration and automation complement each other to produce an automated cloud system.

![Gartner Cloud Adoption](gartner-cloud-adoption.png "width=500")

## As a Service Models

Developers access cloud services in three main models:

- Software as a Service (SaaS)
- Platform as a Service (PaaS)
- Infrastructure as a Service (IaaS)

SaaS is a software licensing and delivery model where a software solution is provided on-demand and hosted by the provider. SaaS solutions often have a subscription fee or freemium pricing model to access the service. The benefit of this approach is that users do not have to install and host the application and can access what they need. Some examples of SaaS solutions are Dropbox and Google Apps like Gmail or Netflix. PaaS platforms give developers a complete cloud development and deployment environment. Developers can load operating systems and development tools on VMs. PaaS provides a contained environment to build cloud applications without needing to manage licensing or underlying application infrastructure. Platforms like Microsoft Azure, Google Cloud Platform, or Amazon Web Services are examples of PaaS. IaaS provides on-demand services to deploy IT infrastructures such as virtual machines, servers, networks, and storage. IaaS is a pay-as-you-go where users pay for the infrastructure they need when they need it. Examples of IaaS platforms include Digital Ocean or AWS EC2. Developers build SaaS systems on IaaS and PaaS platforms, and developers build PaaS platforms on IaaS platforms. As-a-service systems allow developers to achieve cloud orchestration and automation. The diagram below shows how SaaS, IaaS, and PaaS work together to deliver cloud solutions:

<!-- Good image https://azure.microsoft.com/en-au/overview/what-is-paas/ -->

## Benefits of Cloud Orchestration

Cloud orchestration allows developers to automate every part of their cloud solution. Cloud orchestration leads to:

- Increased efficiency
- Cost reductions
- Support for DevOps
- Increased security

When developers automate processes in a cloud solution, the cloud solution can detect when peak times occur and deploy extra services to prevent services from being overloaded. Cloud solutions can also shut down any idle processes that are not needed. The optimized allocation of resources leads to increased efficiency of the platform and reduced costs. Cloud orchestration supports the DevOps framework by allowing continuous integration, monitoring, and testing. Cloud solutions manage all services, which leads to frequent updates and faster troubleshooting. Faster troubleshooting leads to improved security as developers can patch vulnerabilities quickly.

## Cloud Orchestration tools

Terraform is an open-source infrastructure as code tool. You can specify your infrastructure in configuration files to deploy infrastructure on the cloud. Infrastructure as code can be saved and restored between versions. Terraform is a common framework for deploying infrastructure solutions. Kubernetes is a container orchestration tool developed by Google. Containers are lightweight computing units that make up a larger application. Kubernetes works with cloud providers to manage and deploy containers on infrastructure. Resources can be scaled up or down depending on demand which saves costs and increases the reliability of the application.

Many PaaS cloud providers have tools that allow cloud orchestration, such as:

- AWS Cloud Formation
- Microsoft Azure Automation
- IBM Cloud Orchestrator
- Google Cloud Composer

These tools allow developers to automate their cloud environments through infrastructure as code, deployment management GUIs, and integrations to other cloud solutions within the PaaS system. There are dedicated cloud orchestration tools such as:

- RedHat Ansible
- Cloudify
- Morpheus

These dedicated tools provide cloud provisioning, configuration management, and automation. All cloud orchestration tools work with technologies such as Terraform and Kubernetes. The choice of tool will depend on IT budget, preferred languages, the location of preexisting deployments, or other application-specific requirements.

### Where does Octopus Deploy fit in?

Octopus Deploy is a cloud-agnostic deployment tool that makes complex deployments easier. The role of Octopus Deploy in cloud orchestration is to set up and manage the deployment of an application through an on-premise solution or a cloud provider. Octopus takes the application from code to build to deployment. Once the deployment is live, a tool like Kubernetes or AWS Cloud Formation can manage the state of the infrastructure and cloud resources of the application. If developers need to deploy the application with a new version, Octopus can manage the new release and deploy a new release version to the production environment.

## Conclusion

Cloud orchestration is the coordination and automation of workloads, resources, and infrastructure in public and private cloud environments. Cloud orchestration is the automation of the whole cloud system. The benefits of orchestration and automation are increased efficiency, reduced cost, support for DevOps, and increased security. Developers access cloud services in three main models: Software as a service, platform as a service, and infrastructure as a service. These services provide on-demand services for developers to access the resources they need without managing them. Developers can combine SaaS, PaaS, and IaaS to achieve cloud orchestration and automation. Popular frameworks and tools for cloud orchestration include Terraform, Kubernetes, PaaS orchestration tools, and dedicated orchestration tools. Octopus is a deployment tool that can work with your DevOps toolchain to achieve cloud orchestration and happier deployments.
