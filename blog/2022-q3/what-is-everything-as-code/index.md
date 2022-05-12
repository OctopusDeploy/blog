---
title: What is everything as code?
description: A brief summary of the post, 170 characters max including spaces.
author: terence.wong@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: placeholderimg.png
bannerImage: placeholderimg.png
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags:
  - DevOps
  - Cloud Orchestration
  - Everything as Code
---

<!-- see https://github.com/OctopusDeploy/blog/blob/master/tags.txt for a comprehensive list of tags -->

Everything as Code (EaC) is an approach to software development and DevOps that uses code to define and manage IT resources. The code representation of resources makes it easier for developers to audit changes, improve consistency, scale resources, and transfer settings from one environment to another. EaC is ideal where every part of the software life cycle is code. Applications today are far from that ideal, with EaC being used as an umbrella term to cover specific applications of the as-code framework. Infrastructure as code (IaC) and configuration as code (CaC) are popular EaC applications, with other applications like environments as code, data pipelines as code, and machine learning processes as code. This blog will discuss IaC, EaC, some upcoming as-code applications, benefits to EaC, and the future of EaC.


## Infrastructure as Code

IaC is the process of provisioning IT infrastructure through code. Traditionally, developers provisioned IT infrastructure manually. Manual management of IT resources increases the shadow IT footprint of an organization because developers do not provision IT infrastructure in a standard way. There are no controls on who can provision infrastructure and what methods they use.

Terraform is one of the most popular IaC frameworks. Terraform provides a configuration language developers use to define cloud and on-premise resources.

## Configuration as Code

CaC is the process of capturing all system configuration settings in code. CaC was a feature that our users heavily requested. At Octopus Deploy, we released CaC for our platform in 2022 Q1. Our goals were to provide our users with complete CaC functionality without sacrificing usability through the UI. CaC allows our customers to leverage the power of Git in their deployment process through branches, pull requests, and a complete audit log of changes. [We captured some of our design thinking on our blog](https://octopus.com/blog/shaping-config-as-code).

## Other Examples

Environments as Code are another application of EaC, with tools like Docker and Vagrant allowing developers to provision virtual machines and compute environments. Data analytics has applications with EaC, and developers can represent data pipelines and machine learning processes as code. EaC is relevant in any scenario where developers can code processes and resources.

## Benefits

Everything as code allows developers to express IT resources as code. The benefits of this framework are:

- **Consistency:** EaC gives developers more consistency because developers capture infrastructure and configuration settings in a standard framework such as Terraform. This framework reduces human error and improves reliability as the system is version-controlled.
- **Scalability:** EaC improves the scalability of systems because developers capture resource settings in code. Scaling up involves making a small change to a configuration file, and developers can roll any issues back to previous versions.
- **Portability:** EaC allows teams to export their infrastructure, configuration, or other parts of the system and replicate it.
-**Auditability:** EaC makes it easier to audit systems as version control makes changes visible.

## Future of everything as code

EaC is an ideal state where developers represent system parts as code. While many benefits come from this, from an implementation standpoint, it does not always make sense to move a system towards EaC. While deployments as code in Octopus Deploy had significant benefits, there are diminishing returns on converting other platform parts to EaC. There is always an opportunity cost in developing an additional EaC feature compared with other features that organizations could develop in the same timeframe. In practice, organizations should apply EaC where it makes sense and has the most benefit to users.r

## Conclusion

EaC is an approach to software development and DevOps that uses code to define and manage IT resources. EaC has found many applications in IaC, CaC, and other areas of IT. Although EaC is a promising end-state for organizations, this is not always true because there is an opportunity cost to convert parts of a platform to EaC. In the proper context, EaC does have significant benefits providing users with consistency, scalability, portability, and auditability.

Happy deployments!
