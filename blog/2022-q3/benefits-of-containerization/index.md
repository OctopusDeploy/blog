---
title: The benefits of containerization
description: A brief summary of the post, 170 characters max including spaces.
author: terence.wong@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage:
bannerImage:
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags:
  - DevOps
  - Runbooks Series
  - Runbooks
---

<!-- see https://github.com/OctopusDeploy/blog/blob/master/tags.txt for a comprehensive list of tags -->

## What is containerization?

A container is a lightweight, portable computing environment with all the necessary files to run independently. Containerization is the process of making an application runnable as a container. Once the application runs as a container, it will run the same regardless of the infrastructure. Containerization has seen extensive use in recent years. The uptake of containerization has been partly due to the wide adoption of cloud technologies. Cloud environments have allowed container scaling and replication, unlocking their business value. This blog discusses the main benefits of containerization, why it is important for software development and how Octopus Deploy supports containers.

## Benefits

### Containers complement your DevOps process

In our [introduction to DevOps blog](https://octopus.com/blog/introduction-to-devops), we discussed how DevOps as a concept is about removing barriers that get in the way of software delivery. DevOps refines every process between the developer and the customer (flow), has faster feedback loops and continues experimentation and learning. DevOps is a practice that focuses on agility and automation. [Containerization complements microservices, giving applications agility and flexibility](link). Using containerization complements DevOps because software can be deployed and tested faster, improving the feedback loops.

### Containers are portable - build once, run anywhere

Containerization provides application portability. Containers can run anywhere on any infrastructure, such as in the cloud, on a VM, or bare metal. [The Open Container Initiative (OCI)](https://opencontainers.org/) designs open standards for containers. This set of standards ensures that any OCI compliant containers will run the same way on any infrastructure. A container image is a static file that contains executable code to run a process on IT infrastructure. There are container images for several different use cases such as databases, web servers, operating systems, etc. Container image repositories are public access points for container images. Any developer can access them and load a container with an image. The open standards of containers, alongside the wide range of images available, means that developers can load endless services on any infrastructure configuration. [See our list of top container images](link).

### Containers are scalable - allocate resources efficiently

Containers are replicable and run anywhere. PaaS solutions and container orchestration tools like Kubernetes allow developers to operate containers at scale. Container orchestrators can scale individual components in software applications up and down depending on demand and load. Container orchestration leads to cost savings as components do not run for longer than they need to. Scaling improves reliability as container orchestrators can allocate sufficient resources to high-demand parts of the application.

## Who are the primary tools for container technologies?

Cloud PaaS solutions like Microsoft Azure, Amazon Web Services, and Google Cloud Platform have provided the infrastructure to run technologies like Docker and Kubernetes. The Docker container technology was launched as open-source in 2013. Since then, it has gained widespread adoption as the leading container technology. While Docker remains the most popular container technology, it is one part of a larger ecosystem of containers. Kubernetes is a container orchestration technology developed by Google. Companies have been using Kubernetes alongside Docker to manage and scale container solutions.

Docker has been the most common container technology run on Kubernetes. The [v1.24 Kubernetes update](https://kubernetes.io/blog/2022/03/31/ready-for-dockershim-removal/) has deprecated Dockershim - an underlying module providing compatibility between Docker and Kubernetes. The update is mainly due to Docker's compatibility with the Container Runtime Interface. Docker has developed a replacement for Dockershim called cri-dockerd that addresses compatibility issues. A report by Datadog in 2021 indicated a 6% increase in containerd adoption with a correlated dip in Docker usage. The increase in containerd adoption rate may continue as Kubernetes moves away from full Docker support. The containerization and container orchestration landscape is rapidly evolving and changing year to year. The technological tools and popularity may wane and change, but the containerization and container orchestration concepts are here to stay.

## What are the top container images?

According to the Data Dog 2021 report, the top three container images are:

- NGINX

NGINX is an open-source web server, load balancer, and reverse proxy used in several applications. NGINX is the most popular container image, used in over 50% of container environments

- Redis

Redis is an open-source in-memory data structure store used as a database, cache, and message broker. This image is the most popular run image on Kubernetes StatefulSets at around 35%

- Postgres

Postgres is an open-source relational database system that uses the SQL language. Developers used Postgres in container environments, and Kubernetes Stateful sets at around 20%.

Popular container images are often open-source and address a fundamental need in software applications, such as databases, web servers, or caches. These images are maintained and kept up to date by a community and are OCI compliant.


## Containerization support in Octopus Deploy

A deployment process will likely use some form of containers or container orchestration to deploy an application. Octopus Deploy is a deployment management tool that supports containerization. Octopus deploy works with container registries, PaaS providers, Docker, and Kubernetes to provide a best-in-class deployment management tool.

## Conclusion

Containers are standalone computing environments, and containerization converts an application into a runnable container. Containerization gives the development process flexibility and agility, which helps DevOps processes. Containers are highly portable. OCI compliant containers can be built once and run anywhere. With PaaS solutions and container orchestration tools like Kubernetes, containers are scalable to allocate resources efficiently. Containerization is an ever-changing field of research. The popularity of specific tools may shift and change, but Octopus Deploy is container and cloud-agnostic. It works with a range of container registries, PaaS providers, Docker, and Kubernetes to help you make complex deployments easier.


!include <q2-2022-newsletter-cta>

Happy deployments!
