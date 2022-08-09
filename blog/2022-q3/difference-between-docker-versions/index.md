---
title: "Difference between docker.io, docker-cd, and Docker Desktop"
description: Learn which version of Docker to install for your operating system.
author: matthew.casperson@octopus.com
visibility: public
published: 2022-08-09-1400
metaImage: blogimage-gettingstartedcontainerisation-2022.png
bannerImage: blogimage-gettingstartedcontainerisation-2022.png
bannerImageAlt: Man sitting on top of container with green circle with a power up icon
isFeatured: false
tags: 
  - Containers
  - Docker
---

Docker has matured over the years to offer a range of solutions for developers working with containers. This can lead to some confusion, though, as developers need to choose which version of Docker to install. 

In this post, I look at which options are available for which operating systems and offer advice on what choice to make.

## The many facets of Docker

Docker has been part of a large and ongoing movement focused on building, distributing, and running containerized software. While the term "Docker" is often synonymous with containerization, it's worth understanding the various tools and specifications working together to support containerized software.

The core technologies that power Docker are defined by the [Open Container Initiative](https://opencontainers.org/) (OCI), which defines the format for distributable images with the Image Specification (image-spec) and how those images are run with the Runtime Specification (runtime-spec).

The OCI runtime-spec is implemented by a number of [Container Runtimes](https://developers.redhat.com/blog/2018/02/22/container-terminology-practical-introduction#container_engine), including [runc](https://github.com/opencontainers/runc), [crun](https://github.com/containers/crun), and [katacontainers](https://github.com/kata-containers/kata-containers). Container Runtimes perform the low level work required to execute containerized processes.

[Container Engines](https://developers.redhat.com/blog/2018/02/22/container-terminology-practical-introduction#container_engine) such as Docker with [containerd](https://containerd.io/) and [Podman](https://docs.podman.io/en/latest) with [conmon](https://github.com/containers/conmon) are used to pull OCI images and launch containers via a Container Runtime.

The interface between a developer and the software stack used to build, distribute, and run containerized software is technically the Container Engine, and in Docker's case the Container Engine is the `docker` command-line tool.

The combination of a Container Runtime and the Docker Container Engine is referred to as the Docker Engine by the [Docker website](https://docs.docker.com/engine/).

Now we're clearer about what we mean by "Docker", it's time to look at the options available to developers when working with [containerized applications](https://octopus.com/blog/get-started-containers).

## Linux users are spoilt for choice

Containerization technology was first developed in Linux, so it's no surprise that Linux users have choices when it comes to working with containerized applications. 

Docker is not the only option available for Linux users. [Podman](https://docs.podman.io/en/latest/#) provides a drop in replacement for Docker, to the point where the `podman` CLI can be configured as an alias for the `docker` CLI for most developers.

If you want to stick with Docker though, there are 2 options: 

- `docker.io` on Debian/Ubuntu 
- `docker` on Fedora and `docker-ce`

The `docker.io` and `docker` packages are maintained by their respective Linux distributions. They're available to be installed without adding any additional package repositories. They are free and open source.

`docker-ce` is a package provided by Docker. The package is available through a third-party package repository provided for major Linux distributions. Like the `docker.io` and `docker` pacakges, `docker-ce` is free an open source.

There are many discussions on the underlying differences between `docker.io`/`docker` and `docker-ce`. [This question on StackOverflow](https://stackoverflow.com/questions/45023363/what-is-docker-io-in-relation-to-docker-ce-and-docker-ee-now-called-mirantis-k), with close to 90 thousand views, lists some of the pros and cons of each package.

Personally, I install the `docker-ce` package, as it's usually more up to date.

## Docker desktop for everyone else

[Docker Desktop](https://docs.docker.com/desktop/) is the only way to install the Docker Engine on Windows 10 or 11 and macOS operating systems. Docker Desktop is also available for Linux, although Linux users are free to install the Docker Engine separately.

Docker Desktop is a commercial application that [requires payment for some teams](https://docs.docker.com/subscription/#docker-desktop-license-agreement).

## Windows server support

[Windows server 2016 and 2019 include built-in support for running Docker](https://docs.microsoft.com/en-us/virtualization/windowscontainers/about/#the-microsoft-container-ecosystem), but only support running Windows containers.

There's some advice for running Linux containers on Windows server operating systems. [This post on Server Fault](https://serverfault.com/questions/970802/how-to-run-linux-docker-container-on-windows-server-2019/980454#980454) provides a summary and links to other resources. However, none of the information I've seen suggests that Linux containers on Windows server is a supported solution for production environments.

## Conclusion

Developers working on containerized applications first need to install a tool like Docker. But, while the term "Docker" is synonymous with containers, there are many options to choose from, and some don't need any Docker-specific tools at all. 

In this post, we looked at the various tools and specifications that support containerized applications, and noted the options available for developers working with containers.

## Learn more using the Octopus Workflow Builder

If you want to build and deploy containerized applications to AWS platforms such as EKS and ECS, the [Octopus Workflow Builder](https://octopusworkflowbuilder.octopus.com/#/) populates a GitHub repository with a sample application built with GitHub Actions workflows. It configures a hosted Octopus instance with sample deployment projects demonstrating best practices such as vulnerability scanning and Infrastructure as Code (IaC). 

Happy deployments! 
