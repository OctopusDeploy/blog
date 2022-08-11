---
title: "What is cloud orchestration?"
description: The post discusses cloud orchestration and cloud automation, as a service models, cloud orchestration tooling, and where Octopus fits in as a cloud-agnostic deployment tool.
author: terence.wong@octopus.com
visibility: public
published: 2022-08-31-1400
metaImage: blogimage-whatiscloudorchestrationcloudautomation-2022.png
bannerImage: blogimage-whatiscloudorchestrationcloudautomation-2022.png
bannerImageAlt: A cog surrounded by three arrows connected in a circle sits amongst clouds
isFeatured: false
tags:
  - DevOps
  - Cloud Orchestration
---

Wherever you work, you've probably noticed a big shift to the cloud in the applications you use, from storing data like emails or photos, to developing software in a cloud repository like Git. Cloud solutions are dominating the market. A [study by Gartner](https://www.gartner.com/en/newsroom/press-releases/2022-02-09-gartner-says-more-than-half-of-enterprise-it-spending) indicates that revenue for cloud products will overtake traditional IT solutions by 2025. 

If you work with cloud products, you want make them more efficient and cost-effective. Two processes that can help you with that are:

- Cloud orchestration
- Cloud automation 

These concepts are often used interchangeably, but there are some key ways they differ. 

In this post, I cover the differences between cloud orchestration and cloud automation, the various "as a service" models, the benefits of cloud orchestration, tooling, and where Octopus Deploy fits in.

![Gartner Cloud Adoption](gartner-cloud-adoption.png "width=500")

## The difference between cloud orchestration and cloud automation

Cloud orchestration is the coordination and automation of workloads, resources, and infrastructure in public and private cloud environments, and the automation of the whole cloud system. Each part should work together to produce an efficient system. 

Cloud automation is a subset of cloud orchestration focused on automating the individual components of a cloud system. 

Cloud orchestration and automation complement each other to produce an automated cloud system.

## As a service models

Developers access cloud services via 3 main models:

- Software as a Service (SaaS)
- Platform as a Service (PaaS)
- Infrastructure as a Service (IaaS)

SaaS is a software licensing and delivery model where a software solution is provided on-demand and hosted by the provider. SaaS solutions often have a subscription fee or use a freemium pricing model. The benefit of this approach is that you don't have to install and host the application and can access what you need. You probably already use SaaS solutions like Dropbox, Gmail, or Netflix. 

PaaS platforms give you a complete cloud development and deployment environment. You can load operating systems and development tools on VMs. PaaS provides a contained environment to build cloud applications without managing licensing or underlying application infrastructure. Think of the platforms used to build SaaS applications like Microsoft Azure, Google Cloud Platform, or Amazon Web Services.

IaaS provides on-demand services to deploy IT infrastructures such as virtual machines, servers, networks, and storage. IaaS is pay-as-you-go, so you pay for the infrastructure you need when you need it. Think of IaaS as the infrastructure behind PaaS and SaaS systems. Examples of IaaS platforms include Digital Ocean or AWS EC2. 

Developers build SaaS systems on IaaS and PaaS platforms, and developers build PaaS platforms on IaaS platforms. Together, "as a service" systems allow you to achieve cloud orchestration and automation. 

The diagram below shows how SaaS, IaaS, and PaaS work together to deliver cloud solutions:

![As a service models](as-a-service.png "width=500")

## Benefits of cloud orchestration

Cloud orchestration lets you automate every part of your cloud solution. Cloud orchestration leads to:

- Increased efficiency
- Cost reductions
- Support for DevOps
- Increased security

When you automate processes in a cloud solution, the cloud solution can detect when peak times occur and deploy extra services to prevent services from being overloaded. Cloud solutions can also shut down any idle processes that aren't needed. The optimized allocation of resources increases the platform's efficiency and reduces costs. 

Cloud orchestration supports the DevOps framework by allowing continuous integration, monitoring, and testing. Cloud orchestration solutions manage all services, leading to frequent updates and faster troubleshooting. Your applications are more secure as you can patch vulnerabilities quickly.

The journey towards full cloud orchestration is hard to complete. To make the transition more manageable, you can find benefits along the way with cloud automation. For example, you might automate the database component and that speeds up manual data handling, or you install a smart scheduler for your Kubernetes workloads. Even small improvements can save you time and money.

## Cloud orchestration tools

Terraform is an open-source Infrastructure as Code (IaC) tool. You specify your infrastructure in configuration files to deploy infrastructure on the cloud. IaC can be saved and restored between versions. Terraform is a common framework for deploying infrastructure solutions. 

Kubernetes is a container orchestration tool developed by Google. Containers are lightweight computing units that make up a larger application. Kubernetes works with cloud providers to manage and deploy containers on infrastructure. Resources can be scaled up or down depending on demand, which saves costs and increases the reliability of the application.

Many PaaS cloud providers have tools that allow cloud orchestration, such as:

- AWS Cloud Formation
- Microsoft Azure Automation
- IBM Cloud Orchestrator
- Google Cloud Composer

These tools let developers automate their cloud environments through Infrastructure as Code, deployment management GUIs, and integrations to other cloud solutions in the PaaS system. There are also dedicated cloud orchestration tools, such as:

- RedHat Ansible
- Cloudify
- Morpheus

These dedicated tools provide cloud provisioning, configuration management, and automation. All cloud orchestration tools work with technologies such as Terraform and Kubernetes. Your choice of tool will depend on a number of things: 

- IT budget
- Preferred languages
- Location of pre-existing deployments
- Other application-specific requirements

### Where does Octopus Deploy fit in?

Octopus Deploy is a cloud-agnostic deployment tool that helps you set up and manage the deployment of an application through an on-premises solution or a cloud provider. 

Octopus takes the application from code to build to deployment. After the deployment is live, a tool like Kubernetes or AWS Cloud Formation can manage the state of the infrastructure and cloud resources of the application. If you need to deploy the application with a new version, Octopus manages the new release and deploys a new release version to the production environment.

## Conclusion

Cloud orchestration and automation provide you with increased efficiency, reduced cost, support for DevOps, and increased security. 

Cloud services are usually accessed via Software as a Service (SaaS), Platform as a Service (PaaS), and Infrastructure as a Service (IaaS). Saas, PaaS, and IaaS provide on-demand services for you to access resources without managing them. You can combine SaaS, PaaS, and IaaS to achieve cloud orchestration and automation. 

Popular frameworks and tools for cloud orchestration include Terraform, Kubernetes, PaaS orchestration tools, and dedicated orchestration tools. Octopus is a cloud-agnostic deployment tool that can work with your DevOps toolchain to achieve cloud orchestration and faster, more reliable deployments.

Happy deployments!