---
title: The benefits of containerization
description: A post about containerization. The post outlines the main benefits of containerization, lists the top container images, discusses the main containerization technologies and explains how Octopus Deploy works with containerization to make complex deployments easier.
author: terence.wong@octopus.com
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

<!-- see https://github.com/OctopusDeploy/blog/blob/master/tags.txt for a comprehensive list of tags -->

Containers are still a relatively new technology, and to the uninitiated they can seem quite confusing. In this post, I share the benefits of containers, why containers are important for software development, how Octopus Deploy supports containers, and why you might consider adding them to your DevOps processes. 

## What is containerization?

A container is a lightweight, portable computing environment that includes all the necessary files to run independently. 

Containerization is the process of making an application runnable as a container. Once the application can run as a container, it will run the same regardless of the infrastructure that is used to execute the container. Containers are loaded with container images that run a specific application inside the container. The varied number of different container images means that containers have a wide variety of use cases and applications.

Containerization has been widely adopted in recent years, partly due to the availability of cloud technologies, which make container scaling and replication possible, unlocking the business value of containers.  

### Containers complement your DevOps process

In our [introduction to DevOps blog](https://octopus.com/blog/introduction-to-devops), we discussed how DevOps as a concept is about removing barriers that get in the way of software delivery. 

DevOps refines every process between the developer and the customer (flow), encourages faster feedback loops and experimentation and learning. DevOps is a practice that focuses on agility and automation.

Containerization complements DevOps because software can be deployed and tested faster, improving feedback loops. Containerization is also a major factor in the popularity of microservices, a software architecture that improves flexibility and agility.

### Containers are scalable and allocate resources efficiently

Platform as a Service (PaaS) solutions and container orchestration tools like Kubernetes allow developers to operate containers at scale. Container orchestrators can scale individual components in software applications up and down depending on demand and load. This leads to cost savings as components only run for as long they’re needed. Scaling also improves reliability as container orchestrators can allocate sufficient resources to high-demand parts of the application.

### Containers are portable: build once, run anywhere

Because containers are portable, they can run anywhere on any infrastructure, such as in the cloud, on a VM, or bare metal. 

[The Open Container Initiative (OCI)](https://opencontainers.org/) designs open standards for containers, ensuring that any OCI compliant containers will run the same way on any infrastructure. 

To run applications, containers are loaded with container images. A container image is a static file that contains executable code to run a process on IT infrastructure. There are container images for different use cases such as databases, web servers, operating systems, and more. Container image repositories are public access points for container images, which makes them available to developers who can load a container with these images. 

The open standards of containers, alongside the wide range of images available, means that developers can load endless services in their containers to be executed on a variety of infrastructure.

## What are the top container images?

[Docker Hub](https://hub.docker.com/search?q=&type=image) provides a list of popular container images. Some of the top container images are:

- Ubuntu: a Debian-based Linux operating system.

- NGINX: an open-source web server, load balancer, and reverse proxy used in several applications.

- Postgres: an open-source relational database system that uses the SQL language.

- Redis: an open-source in-memory data structure store used as a database, cache, and message broker.

- Alpine: a Linux distribution built around musl libc and BusyBox.

Popular container images are often open-source and address a fundamental need in software applications, such as databases, web servers, or caches. These images are maintained and kept up to date by a community and are OCI compliant. Developers can use a mixture of these container images to build an application, knowing that each container image will run in the same way and on any infrastructure.

## What are the primary tools for container technologies?

Cloud PaaS solutions like Microsoft Azure, Amazon Web Services, and Google Cloud Platform have provided the infrastructure to run technologies like Docker and Kubernetes. The open-source Docker container technology was launched in 2013. Since then, it has gained widespread adoption as the leading container technology. Kubernetes is the most popular container orchestration technology used alongside Docker to manage and scale container solutions.

The containerization landscape is fluid and ever changing. For instance, while Docker has been the most common container technology run on Kubernetes, the [v1.24 Kubernetes update](https://kubernetes.io/blog/2022/03/31/ready-for-dockershim-removal/) deprecated Dockershim - an underlying module that provided compatibility between Docker and Kubernetes. The update was mainly due to Docker’s compatibility with the Container Runtime Interface. Docker has developed a replacement for Dockershim called cri-dockerd that addresses compatibility issues. 

A [report by Datadog](https://www.datadoghq.com/container-report/) in 2021 indicated a 6% increase in containerd adoption with a correlated dip in Docker usage. The increase in containerd adoption rate may continue as Kubernetes moves away from full Docker support. The containerization and container orchestration landscape is rapidly evolving and changing year to year. The technological tools and popularity may wane and change, but the containerization and container orchestration concepts are here to stay.

## Containerization support in Octopus Deploy

A deployment process will likely use some form of containers or container orchestration to deploy an application. Octopus Deploy is a deployment management tool that supports containerization. Octopus Deploy works with container registries, PaaS providers, Docker, and Kubernetes to provide a best-in-class deployment management tool. Regardless of which container technologies are most popular moving forward, Octopus Deploy can work with all of them to provide happier deployments!

## Conclusion

Containers are standalone computing environments, and containerization converts an application into a runnable container. Containerization gives the development process flexibility and agility, which helps DevOps processes. Containers are highly portable, and OCI compliant containers can be built once and run anywhere. With PaaS solutions and container orchestration tools like Kubernetes, containers are scalable to allocate resources efficiently. Containerization is an ever-changing field of research. The popularity of specific tools may shift and change, but Octopus Deploy is container and cloud-agnostic. It works with a range of container registries, PaaS providers, Docker, and Kubernetes to help you make complex deployments easier!


!include <q2-2022-newsletter-cta>

Happy deployments!
