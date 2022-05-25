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

If you work in DevOps or Cloud, you may have worked with tools like GitHub Actions, Jenkins, or Terraform. You may have noticed that these tools all represent parts of the DevOps pipeline as code so that you can store and reuse parts of the code for use later. The code representation of the DevOps pipeline is part of Everything as Code (EaC). But what is EaC and why is it important? 

Everything as Code (EaC) is an approach to software development and DevOps that uses code to define and manage IT resources. The code representation of resources makes it easier for developers to audit changes, improve consistency, scale resources, and transfer settings from one environment to another. Taken literally, EaC is an ideal state where every part of the software life cycle is code. 

Applications today are far from that ideal, with EaC being used as an umbrella term to cover specific applications of the as-code framework. Infrastructure as code (IaC) and configuration as code (CaC) are popular EaC applications, with other applications that cover a range of IT fields. This blog will discuss IaC, CaC, some other as-code applications, benefits to EaC, and the future of EaC.


## Infrastructure as Code

IaC is the process of provisioning IT infrastructure through code. Traditionally, developers provisioned IT infrastructure manually. Manual management of IT resources increases the shadow IT footprint of an organization because developers do not provision IT infrastructure in a standard way. There are no controls on who can provision infrastructure and what methods they use. 

You may have encountered a situation where all your IT infrastructure was under the control of one person (the gate-keeper) in the organization. Only that person understood where everything was stored and how it was configured. With IaC, the contents and location of all infrastructure is transparent and visible to everyone in the organization. Infrastructure is no longer gate-kept by a few people but is version-controlled in a configuration file. Much better!

Terraform is one of the most popular IaC frameworks. Terraform provides a configuration language developers use to define cloud and on-premise resources. Many cloud providers use IaC to allow developers to provision infrastructure on demand in a repeatable way. If you can learn the Terraform language, you can use it in any cloud provider or deployment tool like Octopus to provision infrastructure. 

## Configuration as Code

CaC is the process of capturing all system configuration settings in code. In our platform, the configuration specified the deployment process that users set up in Octopus. CaC was a feature that our users heavily requested. At Octopus Deploy, we released CaC for our platform in 2022 Q1. Our goals were to provide our users with complete CaC functionality without sacrificing usability through the UI. CaC allows our customers to leverage the power of Git in their deployment process through branches, pull requests, and a complete audit log of changes. [We captured some of our design thinking on our blog](https://octopus.com/blog/shaping-config-as-code).

With the release of CaC, our users can choose to use the UI or the source controlled implementation without losing any functionality. A user that only wants to use the UI to make minor changes can work with the power user that wants to use the source controlled implementation to make advanced changes. All functionality that is available to the source controlled implementation is present in the UI version. We believe this keeps our user experience as a first class priority and we are proud of it!

## Other Examples

There are other examples of EaC, some more niche than others, such as:

- **Environments as Code:** Tools that provision computing environments like Docker and Vagrant. Many cloud providers have their own compute environments as code offerings such as Compute Engine by Google and EC2 by Amazon.

- **Data analytics as Code:** Developers can represent data pipelines and machine learning processes as code. This allows data analytics components to be ported from one project to another.

- **DevOps pipelines as Code:** Tools like GitHub Actions and Jenkins represent DevOps pipelines as code. When code is pushed, a process is triggered to built the repository and produce an artefact or deploy a release.

- **Security as Code:** When managing several users in the cloud, security becomes a concern for cloud administrators. Security as code allows security data such as roles and permissions to be represented in a configuration file. Care has to be taken when applying this to sensitive data such as passwords and if sensitive data does need to be as code, encryption techniques must be considered.

EaC is relevant in any scenario where developers can code processes and resources, so it could apply to many more segments of IT. 

## Benefits

Everything as code allows developers to express IT resources as code. The benefits of this framework are:

- **Consistency:** EaC gives developers more consistency because developers capture infrastructure and configuration settings in a standard framework such as Terraform. This framework reduces human error and improves reliability as the system is version-controlled.
- **Scalability:** EaC improves the scalability of systems because developers capture resource settings in code. Scaling up involves making a small change to a configuration file, and developers can roll any issues back to previous versions.
- **Portability:** EaC allows teams to export their infrastructure, configuration, or other parts of the system and replicate it.
- **Auditability:** EaC makes it easier to audit systems as version control makes changes visible.

## Future of everything as code

EaC is an ideal state where developers represent system parts as code. While many benefits come from this, from an implementation standpoint, it does not always make sense to move a system towards EaC. While deployments as code in Octopus Deploy had significant benefits, there are diminishing returns on converting other platform parts to EaC. There is always an opportunity cost in developing an additional EaC feature compared with other features that organizations could develop in the same timeframe. In practice, organizations should apply EaC where it makes sense and has the most benefit to users.

## Conclusion

EaC is an approach to software development and DevOps that uses code to define and manage IT resources. EaC has found many applications in IaC, CaC, and other areas of IT. Although EaC is a promising end-state for organizations, this is not always true because there is an opportunity cost to convert parts of a platform to EaC. In the proper context, EaC does have significant benefits providing users with consistency, scalability, portability, and auditability.

Happy deployments!
