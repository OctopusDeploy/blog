---
title: A beginner's guide to containers
description: A brief overview at containers covering what they are and why they're useful.
author: nikita.dergilev@octopus.com
visibility: public
published: 2023-04-17-1400
metaImage: blogimage-gettingstartedwithdockeralpine2-2022.png
bannerImage: blogimage-gettingstartedwithdockeralpine2-2022.png
bannerImageAlt: Man standing with a laptop in front of a large blue container
isFeatured: false
tags: 
  - DevOps
  - Containers
  - Kubernetes
  - Docker
---

Containers are a popular way to deliver applications. They're well suited to those working in many environments or building software in microservices.

But what are containers exactly? How do containers work? Why should you use them? And how do they differ from related services?

This post explains everything beginners need to know about containers.

## Containers in a nutshell

Containers are lightweight virtual environments. They package everything you need to run an application or microservice, including:

- Code
- Configuration files
- Libraries
- Dependencies

With everything in one deployable format, containers are versatile. You can host, deploy, and move your applications almost anywhere.

Containers are also easy to spin up and tear down, so they're an excellent option for applications that need to scale with demand.

## How containers differ to virtual machines

Virtual machines (VMs) are digital computers that run on servers or other computers. VMs let you install and use common operating systems like Microsoft Windows or Linux. Tech teams commonly use VMs to reduce space taken by physical servers.

Containers, however, are standalone virtual environments without the bloat of an operating system.

These are the key differences between the 2 technologies:

- Isolation - Containers isolate per instance. VMs isolate per operating system.
- Management - You can't change containers after deployment, you can only destroy and replace them. VMs allow for the same changes as physical computers.
- Resources - Containers need fewer resources as they don't emulate operating systems. VMs need extra resources to power their operating systems.
- Portability - You can move containers between hosting environments without major changes to the infrastructure. To move VMs, you must reconfigure your virtual and physical infrastructure.

Both containers and VMs have different strengths and weaknesses. Your choice depends on the needs of your application and the type of environment you're deploying to. 

Containers, however, offer the following benefits:

- Deploy and move your applications anywhere
- Scale your application as demand increases or decreases
- Use fewer resources to run your applications
- Ensure your application and infrastructure are the same across all instances, no matter where you host
- Recover faster due to the speed and ease of spinning 

We went into more depth on [the benefits of containers in a previous post](https://octopus.com/blog/benefits-of-containerization).

## Where Docker and Kubernetes fit alongside containers

If you work in software development, you’ve likely heard of Docker and Kubernetes. Neither is essential for small projects, but together they solve the most common problems with containers.

Let's explore how they help with containers.

### Docker

[Docker](https://www.docker.com/) is a service that helps you package, test, and manage software using Docker's [OCI image format](https://opencontainers.org/). Docker made the OCI format open source, and it's now the industry standard on nearly all container services.

One of Docker's most popular services is its container registry, [Docker Hub](https://hub.docker.com/). Docker Hub lets you store and organize all your container images so they're easy to find and deploy.

### Kubernetes

[Kubernetes](https://kubernetes.io/) is a container orchestration tool that helps you manage and automate the scaling of your software.

When you deploy to Kubernetes, it creates a cluster of containers with your software on your hosting service. Kubernetes can then manage the traffic between your containers based on available resources.

Kubernetes can also create new clusters as container resources hit their limits and destroy them if unused. Kubernetes can also detect broken containers and replace them with new ones.

Both benefits help ensure high availability and consistency no matter which node your customers connect to.

## Conclusion

In this post we explained:

- What containers are
- The benefits and differences between containers and VMs
- What Docker and Kubernetes bring to container management

<!-- Octopus can help simplify container deployments for DevOps teams. [Read more about how Octopus can help you deploy and manage containers](link-to-website-page).-->

For more on containers:

- See everything you need to [get started with containers](https://octopus.com/blog/get-started-containers)
- Follow our guide for [building and deploying a Java app with Docker, Google, Azure, and Octopus](https://octopus.com/blog/deploying-java-app-docker-google-azure)

Happy deployments!