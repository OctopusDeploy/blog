---
title: Containerization - what you need to get started
description: A high-level look at what you need to get started with containerization
author: andrew.corrigan@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: placeholderimg.png
bannerImage: placeholderimg.png
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - DevOps
  - Containers
  - Cloud Orchestration
---

Though not exactly new, containers are becoming the most popular way to run and host applications and microservices. Put simply, containers are lightweight virtual environments that can run apps without the bloat of a full operating system.

Without the bloat, containers have many benefits over traditional infrastructure and virtual machines, including:

- Better security
- Little-to-no system maintenance
- Easy to spin up and tear down
- Easy to scale resources to meet the application's needs

Deployable container images also make it easy to get your app running. Container images usually include your software, all runtimes and prerequisites needed to run your app, plus config set by code. Many companies had their own image formats over the years, but Docker's 'OCI' image (now open-source) quickly became the industry-standard. In fact, many providers use the terms 'OCI images' and 'Docker images' interchangeably.

:::hint
OCI stands for 'Open Container Initiative. The initiative is a container structure that acts as the industry standard format. Most of the major players in tech, development, and cloud services back the initiative and support the OCI format. Read more about it on the [Open Container Initiative website](https://opencontainers.org/).
:::

For those new to containerization's concepts, let's take a high-level look at what you need and how it all fits together.

## Docker Desktop

[Docker Desktop](https://www.docker.com/products/docker-desktop/), available for Windows, Mac, and Linux, is the easiest way to get started with containerization.

It helps you perform the following from your operating system of choice:

- Create, run and test container-ready apps
- Spin up ready-made containerized tools and environments, such as NGINX, MySQL, or Ubuntu
- Manage and send your images to repositories and registries
- Create development environments

Some of these features are in beta or preview at the time of writing.

On Windows, you can switch compatibility between Linux and Windows container images. In most cases, we recommend working with Linux images as few hosting services support native Windows images.

Although Docker Desktop offers a friendly graphical interface, there is a monthly fee for companies with more than 250 employees. Linux users comfortable with command-lines can continue working with containers as they always have.

## Hosting

Obviously, if you want to run your app in a container, you need somewhere to host it for people to access. There are plenty of vendors offering container hosting, and not just the big 3 of [Microsoft Azure](https://azure.microsoft.com/), [Google Cloud](https://cloud.google.com/), and [Amazon Web Services](https://aws.amazon.com/).

You're not limited to cloud services, as most major operating systems support Docker images. So, if you want to host your app on your own hardware, such as a server or your own computer for testing, you can.

Example hosting providers include:

- [Jelastic](https://jelastic.com/docker/)
- [Microsoft Azure](https://azure.microsoft.com/)
- [sloppy.io](https://sloppy.io/en/)
- [Google Cloud](https://cloud.google.com/)
- [Amazon Web Services](https://aws.amazon.com/)

## Container repository

A container repository is where you store your app's images ready for deployment. When deploying your app, your process pulls the latest image from your repository (using your container registry - more on that next) and runs it as a container on your hosting service.

Most hosting services offer private repositories as part of their subscriptions (or for an extra fee). Depending on your hosting vendor, you may get locked into using their repository, so buyer beware.

Example container repositories include:

- [Docker Hub](https://www.docker.com/products/docker-hub/)
- [AWS Serverless Application Repository](https://aws.amazon.com/serverless/serverlessrepo/?nc2=type_a)

## Container registry

A container registry is both a collection of repositories and a searchable catalog used to manage and deploy images.

A registry helps in two important ways:

1. They ensure everyone gets the right versions of your software when they search for it, no matter where they are
2. Deployment processes use the registry to call the correct image from your repository

While there are plenty of registries on the market, they all boast different strengths, such as:  

- Private registries
- Geo-location
- Popular public registries
- On-premises options
- Compatibility with other areas of your pipeline

As with repositories, you may get locked into a particular registry when choosing a hosting provider.

We recently talked about 8 different registry options in another post, but here are a few examples:

- [Docker Hub](https://hub.docker.com/)
- [Harbor](https://goharbor.io/)
- [GitHub Packages](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
- [Red Hat Quay](https://www.redhat.com/en/technologies/cloud-computing/quay)

## Something to deploy (why not try the Octopus Underwater App?)

If you're looking to experiment with containerization but have nothing suitable to deploy, we've got you covered!

The [Octopus Underwater App](https://github.com/OctopusSamples/octopus-underwater-app) is a simple JavaScript app to help you test containerization with different service providers.

![The Octopus Underwater App](/octopus-underwater-app.png)

## Seeing how it all works

Octopus's own Terence Wong wrote an [excellent container deployment guide](https://octopus.com/blog/deploying-java-app-docker-google-azure) in our recent CI series, showing how all these concepts fit together.

His guide walks you through a complete pipeline, including:

- Cloning the Octopus Underwater App's GitHub repository
- Building a Docker image
- Adding the image to Google Cloud's registry and repository services
- Deploying from Google Cloud Registry to an Azure Kubernetes cluster

## What's next?

**Replace this section with our series blurb**

We're covering containerization in-depth over the next few months, covering:  

- blog post
- blog post
- blog post
  
Why not follow the series etc etc?

Happy deployments!