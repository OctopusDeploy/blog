---
title: Octopus Workflow Builder Feedback
description: We are looking for feedback on an early release of a workflow builder.
author: matthew.casperson@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: placeholderimg.png
bannerImage: placeholderimg.png
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - DevOps
  - Containers Series
  - Containers
  - Cloud Orchestration
  - Testing
  - Everything as Code
---

<!-- see https://github.com/OctopusDeploy/blog/blob/master/tags.txt for a comprehensive list of tags -->

What are modern DevOps teams looking for in their continuous delivery workflows? Continuous integration, cloud deployments, feature branching, testing, Software Bill Of Materials (SBOMs), and dependency vulnerability scanning are just a few of the features that high performing teams need to quickly deliver and maintain high quality software.

But how do you *actually* implement these processes? We've shared a lot of opinions in this blog over the years to help teams get the most out of Octopus, but it sometimes felt like we were jumping to the end of the story. This left you, the reader, to build your own sample applications, cloud infrastructure, and Octopus configuration before following along with our latest how-to guide.

We want you to experience the power and joy of a modern continuous delivery workflow, but without spending days setting up your tools. This is why we built the [Octopus Workflow Builder](https://octopusworkflowbuilder.octopus.com/#/), and we'd love your feedback on this early release.

## What is the Octopus Workflow Builder

The [Octopus Workflow Builder](https://octopusworkflowbuilder.octopus.com/#/) provides a simple wizard where you select the platform you want to deploy to (EKS, ECS, and Lambdas are supported in this release), enter the Octopus cloud instance you wish to populate with sample deployment projects, enter your AWS keys, and authorize the application to create a GitHub repository on your behalf.

The [Workflow Builder](https://octopusworkflowbuilder.octopus.com/#/) will then populate a GitHub repository with:

* A sample web application
* Terraform configuration files to create ECR repositories and populate an Octopus space with the Octopus Terraform provider 
* GitHub Action workflows to compile, test, and publish the sample applications and apply the Terraform configuration

The end result is an opinionated CI/CD workflow for deploying, testing, and maintaining cloud based deployments. Best of all, the entire workflow is configured in your GitHub repository and Octopus instance, so you have complete control to customize the process however you want!

The following videos highlight the features of the sample deployment project created by the Workflow Builder:

<iframe width="560" height="315" src="https://www.youtube.com/embed/wABZvJPVCMg" frameborder="0" allowfullscreen></iframe>
<iframe width="560" height="315" src="https://www.youtube.com/embed/vcHdGRS-xzU" frameborder="0" allowfullscreen></iframe>
<iframe width="560" height="315" src="https://www.youtube.com/embed/sex-QLKA5xE" frameborder="0" allowfullscreen></iframe>
<iframe width="560" height="315" src="https://www.youtube.com/embed/Wo4JY8fV_WM" frameborder="0" allowfullscreen></iframe>

## We want your feedback

We would love to get your feedback on this early release of the [Workflow Builder](https://octopusworkflowbuilder.octopus.com/#/) to help us iron out any bugs and to understand if the tool was useful for you.

We have a [GitHub issue](https://github.com/OctopusSamples/content-team-apps/issues/13) where you can submit any feedback.

## Accessing the source code

If you are interested in the source code for the [WorkFlow Builder](https://octopusworkflowbuilder.octopus.com/#/), we've released it on [GitHub](https://github.com/OctopusSamples/content-team-apps).

## Conclusion

We hope you enjoy the [Octopus Workflow Builder](https://octopusworkflowbuilder.octopus.com/#/), and look forward to any feedback you may have to help us make this the best tool it can be.

Happy deployments! 
