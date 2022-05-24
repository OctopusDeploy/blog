---
title: Top 8 container registries
description: There are many container registry services, suitable for all different kinds of teams. We look at the top 8 and why you might consider them.
author: andrew.corrigan@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: placeholderimg.png
bannerImage: placeholderimg.png
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - Containers Series
  - Containers
  - Cloud Orchestration
---

Though they serve different purposes, container registries often get confused for their repository counterparts.

A container repository is storage for your containerized application images. Nowadays, most image repositories focus on the now industry-standard Docker format.

A container *registry*, then, acts as both a collection of container repositories and a searchable catalog where you manage and deploy images.

There are many container registry options on the market, serving customers of different types, sizes and needs. Let's look at our top 8 and why they might appeal.

## Docker Hub

Given Docker is the standard format for container delivery thanks to adoption from all major operating systems, it makes sense that [Docker Hub](https://hub.docker.com/) is also the standard registry for image management. If you're in development, chances are you've already used Docker Hub, especially if you've ever followed one of our technical guides or blogs.

While all registry services on this list help you manage Docker's own image format, Docker Hub as a registry is still worthy of a place here. Not least because it offers a huge public registry you can use to deploy apps from vendors large and small, but to also deliver your own.

## Amazon ECR

[Amazon's Elastic Container Registry (ECR)](https://aws.amazon.com/ecr/) is very useful if already using Amazon Web Services (AWS) to host applications, due to its integration and team management options.

Naturally, Amazon's ECR is woven into all their container hosting services, allowing you to easily manage and push your images to:

- Elastic Container Services (ECS)
- Elastic Kubernetes Services (EKS)
- AWS Lambdas.

(Providing you can get your head around all these similar acronyms, of course.)

Like Docker Hub, their public registry is also worth the price of admission, allowing deployment of many products, free software, and open-source projects. You're never short of existing environments to build on.

## Harbor

[Harbor](https://goharbor.io/) is an open-source registry you can install almost anywhere, but is particularly suited to Kubernetes.

Where certain services tie their registries tightly into their own services, Harbor's freedom makes it a more versatile option. Not only is it compatible with most cloud services and Continuous Integration and Continuous Delivery (CI/CD) platforms, but it's a good on-premises solution too.

## Azure Container Registry

While offering similar services to AWS, Microsoft's [Azure Container Registry (ACR)](https://azure.microsoft.com/en-au/services/container-registry/) not only boasts support for Docker and OCI images, but Helm charts too.

Azure's biggest selling point, however, is its registry geo-replication. This ensures everyone has the same access to images no matter where they're based, and at speeds they're used to.

## GitHub Container Registry

Given GitHub's reach and that it's already available for all users, [GitHub's Container Registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry) (part of a bigger feature called [GitHub Packages](https://github.com/features/packages)) is one of the most approachable options.

There are definite benefits to considering GitHub Packages for container management, including:

- Simplified user management - most of your users already have accounts, after all
- GitHub Actions integration to push, publish and deploy images
- Its cost relative to other services

## Google Container Registry

As other services focus their promises on certain features important to a subset of likely customers, [Google Cloud's Container Registry](https://cloud.google.com/container-registry/) is a solid all-rounder. It does everything you want from a container registry and does it really well.

It's not quite as feature-boastful as AWS or Azure, but Google Cloud's Container Registry is a worthy offering from one of the cloud-provider 'Big 3'.

## Red Hat Quay

Unlike other options, [Red Hat Quay](https://www.redhat.com/en/technologies/cloud-computing/quay) offers private container registries only. This makes it a suitable option for enterprise-level customers in particular.

Free of allegiances to any particular cloud provider, Quay is easy to connect to systems either end of your DevOps pipeline. Like Azure, Quay also includes geo-location options and, interestingly, supports BitTorrent for container distribution. 

Red Hat does also include a pared-down registry solution in their Kubernetes platform, OpenShift. They do, however, recommend Quay for larger teams and organizations.

## What's next

**Replace this section with our series blurb**

We're covering containerization in-depth over the next few months, covering:  

- blog post
- blog post
- blog post
  
Why not follow the series etc etc?

Happy deployments! 