---
title: Difference between docker.io, docker-cd, and Docker Desktop
description: Learn which version of Docker to installed for your operating system
author: matthew.casperson@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: blogimage-gettingstartedcontainerisation-2022.png
bannerImage: blogimage-gettingstartedcontainerisation-2022.png
bannerImageAlt: Man sitting on top of container with green circle with a power up icon
isFeatured: false
tags: 
  - Containers
---

Docker has matured over the years to offer a range of solutions for developers working with containers. This can lead to some confusion as developers must choose which version of Docker to install, and in this post we'll look at what options are available for which operating systems, along with advice on what choice to make.

## The many facets of Docker

Docker has been part of a large and ongoing movement focusing on building, distributing, and running containerized software. While the term "Docker" is often synonymous with containerization, it is worth understanding the various tools and specifications that work together to support containerized software.

The core technologies that power Docker are defined by the [Open Container Initiative](https://opencontainers.org/) (OCI), which defines the format for distributable images, with the Image Specification (image-spec) and how those images are run with the Runtime Specification (runtime-spec).

The OCI runtime-spec is implemented by a number of [Container Runtimes](https://developers.redhat.com/blog/2018/02/22/container-terminology-practical-introduction#container_engine), including [runc](https://github.com/opencontainers/runc), [crun](https://github.com/containers/crun), and [katacontainers](https://github.com/kata-containers/kata-containers)). Container Runtimes perform the low level work required to execute containerized processes.

[Container Engines](https://developers.redhat.com/blog/2018/02/22/container-terminology-practical-introduction#container_engine) such as Docker with [containerd](https://containerd.io/) and [Podman](https://docs.podman.io/en/latest) with [conmon](https://github.com/containers/conmon) are used to pull OCI images and launch containers via a Container Runtime.

The interface between a developer and the software stack used to build, distribute, and run containerized software is technically the Container Engine, and in Docker's case the Container Engine is the `docker` command line tool.

The combination of a Container Runtime and the Docker Container Engine is referred to as the Docker Engine by the [Docker website](https://docs.docker.com/engine/).

Now that we have a clearer idea of what we mean when talking about "Docker", it is time to look at the options available to developers when working with containerized applications.

## Linux users are spoilt for choice

Containerization technology was first developed in Linux, so it is no surprise that Linux users have choices when it comes to working with containerized applications. 

Docker is not the only option available for Linux users. [Podman](https://docs.podman.io/en/latest/#) provides a drop in replacement for Docker, to the point where the `podman` CLI can be configured as an alias for the `docker` CLI for most developers.

For those that want to stick with Docker though, there are two options: `docker.io` on Debian/Ubuntu or `docker` on Fedora and `docker-ce`.

The `docker.io` and `docker` packages are maintained by their respective Linux distributions. They are available to be installed without adding any additional package repositories. They are free and open source.

`docker-ce` is a package provided by Docker. The package is available through a third party package repository provided for the major Linux distributions. Like the `docker.io` and `docker` pacakges, `docker-ce` is free an open source.

There are many discussions on the underlying differences between `docker.io`/`docker` and `docker-ce`. [This question on StackOverflow](https://stackoverflow.com/questions/45023363/what-is-docker-io-in-relation-to-docker-ce-and-docker-ee-now-called-mirantis-k), with close to 90 thousand views, lists some of the pros and cons of each package.

Personally, I install the `docker-ce` package, as it is usually more up to date.

## Docker desktop for everyone else

[Docker Desktop](https://docs.docker.com/desktop/) is the only way to get the Docker Engine on Windows 10 or 11 and macOS operating systems. Docker Desktop is also available for Linux, although Linux users are free to install the Docker Engine separately.

Docker Desktop is a commercial application that [requires payment for some teams](https://docs.docker.com/subscription/#docker-desktop-license-agreement).

## Windows containers

[Windows server 2016 and 2019 include native support for running Docker](https://docs.microsoft.com/en-us/virtualization/windowscontainers/about/#the-microsoft-container-ecosystem), but only support running Windows containers.

There is some advice available for running Linux containers on Windows server operating systems. [This post on Server Fault](https://serverfault.com/questions/970802/how-to-run-linux-docker-container-on-windows-server-2019/980454#980454) provides a summary and links to other resources. However, none of the information I have seen suggests that Linux containers on Windows server is a supported solution for production environments.

## Conclusion

Developers working on containerized applications will first need to install a tool like Docker. But, while the term "Docker" is synonymous with containers, there are many options to choose from, and some do not require any Docker specific tools at all. In this post we looked at the various tools and specifications that support containerized applications, and noted the options available for developers needing to work with containers.

## Learn more

If you are looking to build and deploy containerized applications to AWS platforms such as EKS and ECS, the [Octopus Workflow Builder](https://octopusworkflowbuilder.octopus.com/#/) populates a GitHub repository with a sample application built with GitHub Actions workflows and configures an Hosted Octopus instance with sample deployment projects demonstrating best practices such as a vulnerability scanning and Infrastructure as Code (IaC). 

Happy deployments! 
