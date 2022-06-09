---
title: Octopus Workflow Builder feedback
description: Request for feedback on the early release of a workflow builder.
author: matthew.casperson@octopus.com
visibility: private
published: 2022-06-14
metaImage: blogimage-feedback_2021_01.png
bannerImage: blogimage-feedback_2021_01.png
bannerImageAlt: Octopus employee at laptop with headset and icons representing customer feedback
isFeatured: false
tags: 
  - DevOps
  - Containers
  - Cloud Orchestration
---

What do modern DevOps teams demand from their continuous delivery workflows? Continuous integration, cloud deployments, feature branching, testing, Software Bill Of Materials (SBOMs), and dependency vulnerability scanning are just a few of the features that high performing teams need to quickly deliver and maintain high quality software.

But how do you *actually* implement these processes? We've shared a lot of opinions in this blog over the years to help teams get the most out of Octopus, but it feels like we were often telling the end of the story. We wanted to "make complex deployments easy", but we frequently skipped over the unglamorous work of setting up even the most minimal realistic sample environment. This left you, the reader, to write your own sample applications, instantiate your own cloud infrastructure, populate your own Octopus configuration, and copy/paste our samples before following along with our latest how-to guide.

We want you to experience the power and joy of a modern continuous delivery workflow to the platforms you actually use, but without spending days setting up your tools. This is why we built the [Octopus Workflow Builder](https://octopusworkflowbuilder.octopus.com/#/), and we'd love your feedback on this early release.

## What is the Octopus Workflow Builder

The [Octopus Workflow Builder](https://octopusworkflowbuilder.octopus.com/#/) provides a simple wizard where you select the platform you want to deploy to (EKS, ECS, and Lambdas are supported in this release), enter the Octopus cloud instance you wish to populate with sample deployment projects, enter your AWS keys, and authorize the application to create a GitHub repository on your behalf.

The [Workflow Builder](https://octopusworkflowbuilder.octopus.com/#/) will then populate a GitHub repository with:

- Sample web applications
- Terraform configuration files to create ECR repositories and populate an Octopus space with the [Octopus Terraform provider ](https://registry.terraform.io/providers/OctopusDeployLabs/octopusdeploy/latest/docs)
- GitHub Action workflows to compile, test, and publish the sample applications and apply the Terraform configuration

This in turn populates your Octopus instance with:

- Environments, accounts, lifecycles, feeds, and targets
- An infrastructure deployment project creating either an EKS cluster, and ECS cluster, or an API Gateway to expose Lambdas
- A backend deployment project that deploys a RESTful API, smoke tests it, and performs an integration test with Postman
- A frontend deployment project that deploys a static web application, smoke tests it, and performs an end-to-end test with Cypress
- An environment where security scanning is performed against an SBOM package generated from the associated application's dependencies

The end result is an opinionated CI/CD workflow for deploying, testing, and maintaining real cloud-based applications. Best of all, the entire workflow is configured in your own GitHub repository and Octopus instance, so you have complete control to customize the process however you want.

The following videos highlight the features of the sample deployment project created by the Workflow Builder:

<iframe width="560" height="315" src="https://www.youtube.com/embed/wABZvJPVCMg" frameborder="0" allowfullscreen></iframe>
<iframe width="560" height="315" src="https://www.youtube.com/embed/vcHdGRS-xzU" frameborder="0" allowfullscreen></iframe>
<iframe width="560" height="315" src="https://www.youtube.com/embed/sex-QLKA5xE" frameborder="0" allowfullscreen></iframe>
<iframe width="560" height="315" src="https://www.youtube.com/embed/Wo4JY8fV_WM" frameborder="0" allowfullscreen></iframe>

## We want your feedback

We would love to get your feedback on this early release of the [Workflow Builder](https://octopusworkflowbuilder.octopus.com/#/) to help us iron out any bugs and to understand if the tool is useful for you. 

We have a [GitHub issue](https://github.com/OctopusSamples/content-team-apps/issues/13) where you can submit any feedback.

## Accessing the source code

If you're interested in the source code for the [WorkFlow Builder](https://octopusworkflowbuilder.octopus.com/#/), we released it on [GitHub](https://github.com/OctopusSamples/content-team-apps).

## Conclusion

We hope you enjoy the [Octopus Workflow Builder](https://octopusworkflowbuilder.octopus.com/#/), and look forward to any feedback you have to help us make this the best tool it can be.

<span><a class="btn btn-success" href="https://github.com/OctopusSamples/content-team-apps/issues/13">Provide feedback</a></span>

Happy deployments! 
