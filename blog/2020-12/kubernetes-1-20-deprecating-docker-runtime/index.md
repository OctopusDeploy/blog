---
title: Kubernetes 1.20 is deprecating Docker Runtime - What's happening?
description: With the news out that Kubernetes 1.20 is deprecating Docker, there has been a lot of panic. This blog post is all about what's happening and what you can do to solve the problem.
author: michael.levan@octopus.com
visibility: private
published: 2050-12-01
metaImage:
bannerImage:
tags:
 - DevOps
 - Kubernetes
---

Over the course of the week (11/30/2020-12/04/2020) and surely more to come, there has been a lot of questions and a lot of concerns around Kubernetes dropping support for Docker.

The problem is, that's not the case. Kubernetes isn't dropping support for Docker. They're dropping runtime support for the Docker runtime. You have to remember, Docker is an entire stack, not just a container or a container runtime. There are several components that go into Docker.

## Don't panic

I promise you that this is not as crazy or insane as it sounds. In fact, this entire thing has been planned for years now. If you're a user of Kubernetes, for example, you're using Azure Kubernetes Service (AKS) or Elastic Kubernetes Service (EKS), not a lot is changing for you. It's pretty much going to continue to be the same. For example, if you upgrade the Kubernetes API version in AKS to 1.19, then you're already running [containerd](https://containerd.io/) (we'll get into containerd in a second), which is a runtime, like the Docker runtime.

All of the things that you're currently using to build containers and images will still work, including:

- [Dockerfiles](https://docs.docker.com/engine/reference/builder/) - yes, you can still build **Docker** images and run the containers in Kubernetes
- [Docker Compose](https://docs.docker.com/compose/) - yes, it'll still work on your local Docker instances
- [Dockerhub](https://hub.docker.com/) - yes, Dockerhub will still exist. Remember, Docker is a huge stack. Docker images are just one part of that stack.
- Other Docker registries - yes, they'll still work. The way you build, store, and maintain Docker images stays the same.

## Kubernetes 1.20 is deprecating Docker - What's actually happening?

Let's get into what's happening and why it's happening.

Docker in itself was a monolith and that's how it was built. Because they wanted to move into a modern application approach, various things were changed. The one component you're most likely interested is, as you may think, the runtime. Instead, it's actually [Dockershim](https://godoc.org/k8s.io/kubernetes/pkg/kubelet/dockershim).

Dockershim implements a container runtime interface for Docker integration using Kubernetes. However, it was always a goal to move away from Dockershim (hence, the "shim" in the name). It was created at first to help implement integration with Kubernetes, but it ended up just creating an extra hop. Because of that, Docker started working on Containerd.

Containerd, much like [CRI-O](https://www.redhat.com/en/blog/introducing-cri-o-10#:~:text=CRI%2DO%3A%20A%20Lightweight%20Container%20Runtime%20for%20Kubernetes&text=The%20name%20derives%20from%20CRI,support%20any%20OCI%2Dconformant%20runtime.), is a container runtime that is part of the Open Container Iniaitive ([OCI](https://opencontainers.org/)). 

Kubernetes maintaining Dockershim was becoming a huge weight on their shoulders. With that comes the CRI standard. The CRI standard was created to lift the weight of their shoulders. It allows for a very smooth transition for interoperability for container runtimes. 

Now, here's the problem. Docker doesn't implement CRI, which is why the Docker runtime is being deprecated. However, that's okay, because that was sort of the plan.

## Kubernetes is deprecating Dockershim - This was planned

The idea of getting rid of Dockershim was always the plan. There wasn't much of a reality where it could stay permanent because Dockershim required that extra hop/layer, which simply wasn't efficient.

Before docker started working on Containerd, they had Dockershim. Dockershim was a way to implement CRI support for Docker. Because of that extra hop, it didn't make sense to keep it around. That's why Docker started focusing on Containerd integration.

So, in short, this was several years in the making and was planned. Dockershim was simply a place-holder.

## What happens in Kubernetes 1.20

The Docker runtime will officially be deprecated starting in Kubernetes API version 1.20. If you still have the Docker runtime, that's okay. Although, you should very-much be starting to think about moving to another runtime.

Starting in 1.20, there will be warning log printed at the kubelet startup if you're still using the Docker runtime. That's it, just a warning message.

As of right now, the absolutely earliest Kubernetes will ship without Dockershim will be version 1.23 in late 2021. So, in short, you have roughly one year or so to sort this out.

## Who's Impacted

With all of the change going on, let's break down who's impacted.

First, if you're writing code, containerizing code, and shipping it, you're not going to be impacted. In fact, you probably won't even see or feel a difference. Everything is smooth-sailing for you.

If you're a Kubernetes cluster-admin, you will be impacted, but not in a terrible way. All you'll have to do is replace the Docker runtime with CRI-O, Containerd, or another CRI compliant runtime.

Here's a great post and break-down of how to implement Containerd.

[https://kubernetes.io/docs/setup/production-environment/container-runtimes/#cri-o](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#cri-o)

## Closing Thoughts
Luckily, this implementation and change is much less severe than what everyone originally expected. As a developer, you'll hardly notice the difference. As an operations person, you'll have to do an update to another container runtime other than the Docker runtime. This change is a step in the right direction and removes the extra hop with Dockershim. Now, Kubernetes has one less thing to maintain and the product is faster for it.
