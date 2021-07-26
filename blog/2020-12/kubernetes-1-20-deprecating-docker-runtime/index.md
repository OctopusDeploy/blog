---
title: "Kubernetes 1.20 is deprecating Docker Runtime: What does this mean?"
description: With the news that Kubernetes 1.20 is deprecating Docker, there has been a lot of panic. This blog post explains what's happening and what you can do to solve the problem.
author: michael.levan@octopus.com
visibility: public
published: 2020-12-14
metaImage: kubernetes-1-20-deprecating-docker-runtime.png
bannerImage: kubernetes-1-20-deprecating-docker-runtime.png
bannerImageAlt: Kubernetes 1.20 is deprecating Docker Runtime What does this mean?
tags:
 - DevOps
 - Kubernetes
---

![Kubernetes 1.20 is deprecating Docker Runtime: What does this mean?](kubernetes-1-20-deprecating-docker-runtime.png)

Over the course of the week between November 11th and December 4th, there were a lot of questions and concerns about Kubernetes dropping support for Docker.

The problem is, that's not the case. Kubernetes isn't dropping support for Docker. They're dropping runtime support for the Docker runtime. You have to remember, Docker is an entire stack, not just a container or a container runtime. There are several components that go into Docker.

## Don't panic

I promise you that this is not as crazy as it sounds. In fact, this entire thing has been planned for years now. If you're a user of Kubernetes, for example, Azure Kubernetes Service (AKS) or Elastic Kubernetes Service (EKS), not a lot is changing for you. It's pretty much going to continue as before. For example, if you upgrade the Kubernetes API version in AKS to 1.19, you're already running [Containerd](https://containerd.io/), which is a runtime, like the Docker runtime.

All of the things that you're currently using to build containers and images will still work, including:

- [Dockerfiles](https://docs.docker.com/engine/reference/builder/): You can still build **Docker** images and run the containers in Kubernetes.
- [Docker Compose](https://docs.docker.com/compose/): It will still work on your local Docker instances.
- [Dockerhub](https://hub.docker.com/): Dockerhub will still exist. Remember, Docker is a huge stack. Docker images are just one part of that stack.
- Other Docker registries: They will still work. The way you build, store, and maintain Docker images stays the same.

## Kubernetes 1.20 is deprecating Docker - What's actually happening?

Let's get into what's happening and why it's happening.

Docker was built as a monolith, but things are changing to a more modern application approach. The runtime is provided by [Dockershim](https://godoc.org/k8s.io/kubernetes/pkg/kubelet/dockershim), and that's one of the things that's changing and it's causing a lot of the confusion.

Dockershim implements a container runtime interface for Docker integration using Kubernetes. However, it was always a goal to move away from Dockershim (hence, the "shim" in the name). It was initially created to help implement integration with Kubernetes, but it ended up just being an extra hop. Because of that, Docker started working on Containerd.

Containerd, much like [CRI-O](https://www.redhat.com/en/blog/introducing-cri-o-10#:~:text=CRI%2DO%3A%20A%20Lightweight%20Container%20Runtime%20for%20Kubernetes&text=The%20name%20derives%20from%20CRI,support%20any%20OCI%2Dconformant%20runtime.), is a container runtime that is part of the Open Container Initiative ([OCI](https://opencontainers.org/)). 

Kubernetes maintaining Dockershim was becoming a huge weight on their shoulders because Dockershim was an extra hop to get to the runtime in Kubernetes. Because of that, Kubernetes decided to go towards using CRI, which allows for a very smooth transition for interoperability for container runtimes. That means, no more _hop_ is needed.

Now, here's the problem. Docker doesn't implement CRI, which is why the Docker runtime is being deprecated.

## Kubernetes is deprecating Dockershim - This was planned

Getting rid of Dockershim was always the plan. There wasn't much of a reality where it could permanently stay because Dockershim required that extra hop, which simply wasn't efficient, and it was really just more of a place-holder, while they worked on Containerd.

## What happens in Kubernetes 1.20

The Docker runtime will officially be deprecated starting in Kubernetes API version 1.20. If you still have the Docker runtime, that's okay. Although, you should start thinking about moving to another runtime.

Starting in 1.20, if you're still using the Docker runtime, there will be a warning log printed at the kubelet startup. That's it, just a warning message.

As of right now, the absolute earliest Kubernetes will ship without Dockershim is version 1.23 in late 2021. In short, you have roughly one year or so to prepare.

## Who's impacted

With all of the change going on, let's break down who's impacted.

First, if you're writing code, containerizing code, and shipping it, you're not going to be impacted. In fact, you probably won't even see or feel a difference. Everything is smooth sailing for you.

If you're a Kubernetes cluster admin, you will be impacted, but not in a terrible way. All you have to do is replace the Docker runtime with CRI-O, Containerd, or another CRI compliant runtime.

Here's a great post and break-down of how to implement Containerd: [https://kubernetes.io/docs/setup/production-environment/container-runtimes/#cri-o](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#cri-o).

## Closing Thoughts

Luckily, this change is much less severe than what everyone originally expected. As a developer, you'll hardly notice the difference. As an operations person, you'll have to update to a container runtime other than the Docker runtime. This change is a step in the right direction and removes the extra hop with Dockershim. Now, Kubernetes has one less thing to maintain and the product is faster for it.
