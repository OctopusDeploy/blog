---
title: Resources to get started migrating to the cloud
description: This blog provides some resources to gettings started with migrating to the cloud. We supply some whitepapers, free tools, external reports, and Octopus authored blogs and guides to help users be informed about the cloud and get started.
author: terence.wong@octopus.com
visibility: private
published: 2023-08-24-1400
metaImage: blogimage-whatiscloudorchestrationcloudautomation-2022.png
bannerImage: blogimage-whatiscloudorchestrationcloudautomation-2022.png
bannerImageAlt: A cog surrounded by three arrows connected in a circle sits amongst clouds
isFeatured: false
tags:
  - DevOps
  - Cloud Orchestration
---

Cloud-native technologies have emerged as a way for companies to gain a competitive advantage through scalability, economies of scale, and global reach. Migrating to the cloud and modernizing systems are important issues for most IT professionals. Despite the clear need, there can be some challenges to knowing where to start.

At Octopus, we meet this challenge by providing some resources for getting started with cloud migration. These resources are a mix of Octopus-developed tools, whitepapers, guides, and external reports. This list of resources exposes you to a wide variety of sources so you can have an informed opinion about the benefits of the cloud and get practical tools to get started.

## Whitepapers

We want to provide in-depth research about important issues to our customers and us. As a cloud-native dedicated continuous delivery tool, we want to help our users understand continuous delivery and cloud-native technologies. To achieve this, we have developed four whitepapers to help inform our readers and give them actionable steps to get started.

### Migrating to the cloud

The migrating to the cloud whitepaper provides an argument for migrating to the cloud and the first steps to take. It contains researched-backed reports and distills cloud migration into a series of phases: strategy, implementation, and operation. The whitepaper provides straightforward, actionable steps your company can take when migrating to the cloud.

### Importance of continuous delivery

The importance of continuous delivery whitepaper provides a deep dive into the best practice principles, benefits, and why continuous delivery's technical capabilities are fundamental to a successful DevOps adoption. The white paper includes case studies and discussions on challenges your company may face when implementing continuous delivery.

### How to map deployment pipelines

The deployment pipeline is the key ingredient for Continuous Delivery. By mapping and improving the deployment pipeline, you can increase the frequency of deployments while reducing your change-related risk. High-performing teams use Continous Delivery to drive their software delivery and achieve higher levels of organizational performance.

In this paper you will learn how to apply Lean thinking to start from where you are, mapping and improving your deployment pipeline while planning your adoption of the core technical practices.

### Measuring continuous delivery and DevOps

You might be introducing new practices and capabilities as part of Continuous Delivery and DevOps adoption, or making changes as part of continuous improvement. In either case, you need a way to tell if changes are improving your ability to deliver software and, ideally, if they help your organization achieve its goals.

In this white paper, you find several approaches for measuring your progress. There are statement-based assessments and metric-driven measurements. You can use them at different times or combine them to create a custom view of your organization.

<!--![Importance of continuous delivery whitepaper cover](importance-of-continuous-delivery-white-paper.png)-->

## Octopus free tools

While supporting our customers, we see that customers would like to set up deployment pipelines, but there are common barriers to getting started. Often, customers have a repository they want to deploy. To do this, they must perform scaffolding steps to support the deployment. These steps require knowledge of cloud platforms, containers, image repositories, infrastructure as code (IaC), and more. Extra scaffolding locks out a portion of our user base who don't have the time to learn these technologies and want a simple deployment with a few clicks of a button. To address these issues, we have developed free tools to help remove some barriers to getting started with modern CI/CD pipelines.

### Workflow Builder

The Octopus workflow builder was born out of a need to get users started quickly in a cloud CI/CD workflow. Users provide a GitHub repository and credentials, and the workflow builder will automatically set up a deployment pipeline, managed by Octopus and deployed to a cloud platform.

![Workflow Builder](workflowbuilder.png "width=500")

### Kubernetes YAML generator

When working with Kubernetes, you must supply a YAML configuration file for your deployments. The Kubernetes YAML generator is a UI-based tool that lets you quickly generate Kubernetes-compliant YAML code. This code can be pasted directly into the Octopus UI to help you get started quickly with Kubernetes deployments. [If you would like to try, we wrote a blog on how to get started and use the Kubernetes YAML generator.](https://octopus.com/blog/octopus-kubernetes-yaml-generator)

![YAML Generator](yaml-generator.png "width=500")

## External reports

When validating your cloud-native approach, aligning with the broader cloud community is helpful. If you are wanting to convince senior management to migrate to the cloud, you will need a research backed statistics to valudate your argument. External reports provide surveys of cloud usage, best practices, and use cases that you can use to back up your strategies and gain momentum for cloud migration in your company.

## State of the cloud reports

State of the cloud reports gives a snapshot of the significant trends toward cloud-native. The main trends in these reports are that cloud usage has increased, and the trend is increasing towards the future. Companies have to modernize their systems and workforce to maintain a competitive advantage.

- [Hashicorp](https://www.hashicorp.com/state-of-the-cloud)
- [Flexera](https://resources.flexera.com/web/pdf/Flexera-State-of-the-Cloud-Report-2022.pdf)
- [Konveyor](https://www.konveyor.io/modernization-report/?utm_source=thenewstack&utm_medium=website&utm_campaign=platform)
- [Foundary](https://resources.foundryco.com/download/cloud-computing-executive-summary)

## Best-practice reports

To migrate to the cloud, it's helpful to understand the best practices that major cloud providers have used to become cloud-native. These reports suggest that planning is essential, and companies face implementation and operational concerns. When moving towards cloud-native, hybrid approaches are recommended to bridge the gap between traditional systems and complete cloud-native solutions.

- [Amazon](https://pages.awscloud.com/rs/112-TZM-766/images/AWS_Migration_8_Best_Practices_ebook_final.pdf)
- [Microsoft](https://azure.microsoft.com/en-au/migration/migration-journey/#how-to-migrate)
- [Google](https://cloud.google.com/architecture/migration-to-gcp-getting-started)

## Octopus supporting resources

At Octopus, we have authored several blogs and guides that can help you with cloud migration. We focus on educating our users on the best practices of cloud-native deployments and giving them practical guides on how to get started.

### The ten pillars of pragmatic kubernetes deployments

Kubernetes is a popular container orchestration tool. It is open source and works on all the major cloud platforms. Despite its popularity, it can be intimidating to learn and know where to get started. To help our users use Kubernetes, we wrote an [eBook on getting started with Kubernetes](https://github.com/OctopusDeploy/TenPillarsK8s/releases/tag/0.1.269-main). The eBook takes users through the ten pillars of pragmatic Kubernetes deployments:

1. Repeatable deployments
1. Verifiable deployments
1. Seamless deployments
1. Recoverable deployments
1. Visible deployments
1. Measurable deployments
1. Auditable deployments
1. Standardized deployments
1. Maintainable deployments
1. Coordinated deployments 

The guide contains practical steps that users can follow in an Octopus environment to set up their Kubernetes deployments. To support this guide, we wrote an [ultimate guide to Kubernetes microservice deployments.](https://octopus.com/blog/ultimate-guide-to-k8s-microservice-deployments)

![Ten Pillars Cover](Kubernetescover.png)

### The DevOps engineer handbook

Our DevOps engineer handbook provides you with a central place to learn about all things DevOps. We provide resources on common questions, definitions and debates in the field of DevOps. The handbook is a good tool to get up to speed with the latest trends in the DevOps space.

### Other relevant blog posts

We have created several blogs that are relevant to cloud-native technologies. Some of these blogs are educational about the benefits of the cloud, and others are instructional, guiding you through how to set up a cloud-native workflow:

1. [What is cloud orchestration?](https://octopus.com/blog/what-is-cloud-orchestration)
1. [Microservices and frameworks](https://octopus.com/blog/microservices-and-frameworks)
1. [Monolith versus microservices](https://octopus.com/blog/monoliths-vs-microservices)
1. [The benefits of containerization](https://octopus.com/blog/benefits-of-containerization)
1. [Containerization - what you need to get started](https://octopus.com/blog/get-started-containers)
1. [We have an entire series on continuous integration (CI) which gives practical examples of CI servers working in cloud-native technologies](https://octopus.com/blog/tag/CI%20Series)


## Conclusion

Research and surveys have shown that cloud technologies provide a clear advantage over traditional systems. To help our users get started with cloud migration, we have provided whitepapers, free tools, external reports, and Octopus-published blogs and guides. These resources help give you a base understanding of cloud technologies and some practical tools and steps to start your migration efforts. We hope that these resources are useful to you, and that they help you get started on your cloud migration journey!
