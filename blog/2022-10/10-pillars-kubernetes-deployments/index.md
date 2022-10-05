---
title: 10 pillars of pragmatic Kubernetes deployments
description: The 10 pillars of pragmatic Kubernetes deployments is a comprehensive guide to building a repeatable deployment pipeline from Octopus to Kubernetes. Download the ebook. 
author: matthew.casperson@octopus.com
visibility: public
published: 2022-10-19-1400
metaImage: imgblog-tenpillarspragmatickubernetesdeployment-2022.ai.png
bannerImage: imgblog-tenpillarspragmatickubernetesdeployment-2022.ai.png
bannerImageAlt: An open book sits in front of a Kubernetes-branded building with ten columns
tags:
  - DevOps
  - Kubernetes
---

Following on from the [10 pillars of pragmatic deployments](https://octopus.com/blog/ten-pillars-of-pragmatic-deployments), Matthew Casperson wrote an ebook about the [10 pillars of pragmatic **Kubernetes** deployments](https://github.com/OctopusDeploy/TenPillarsK8s/releases/latest) with Octopus Deploy.

The 10 pillars speak to the needs of modern DevOps teams, always being asked to deliver more in less time. By understanding the value of each pillar, and learning practical implementations, DevOps teams can meet these challenges head-on.


## Who will benefit from the ebook?

The ebook is a comprehensive guide to building a repeatable deployment pipeline from Octopus to Kubernetes. 

It’s aimed at people looking to build deployment pipelines that scale as the following increases:

- The number of teams involved in deployments
- The number of applications being deployed
- The frequency of deployments
- The expectation that code reaches customers faster


## What you need to get started

To follow along with the exercises in the book, you need: 

- A Kubernetes cluster. The book uses a Kubernetes cluster hosted in Google Cloud, but beyond initial administration tasks, which rely on the administrative credentials exposed by a Google Cloud Kubernetes cluster, any Kubernetes cluster can be used.
- Access to administrative credentials. 
- An instance of Octopus. If you're not an Octopus customer, you can sign up for a [free trial](https://octopus.com/start). Trial licenses are valid for 30 days with unlimited targets, so you can follow the examples in the book.

The ebook assumes you know concepts like pods, deployments, services, service accounts, secrets, and configmaps. However, all the Kubernetes resources created in the book provide detailed instructions on the values to enter into Octopus, so it's possible to get a sense of the deployment processes without a deep understanding of every setting. 

<span><a class="btn btn-success" href="https://github.com/OctopusDeploy/TenPillarsK8s/releases/latest">Download the ebook</a></span>

Happy deployments!