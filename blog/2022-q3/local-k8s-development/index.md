---
title: Testing Kubernetes locally
description: Learn about the tools available for developers testing Kubernetes on their local machines. 
author: matthew.casperson@octopus.com
visibility: public
published: 2022-08-17-1400
metaImage: blogimage-testingkubernetes-2022.png
bannerImage: blogimage-testingkubernetes-2022.png
bannerImageAlt: Kubernetes logo on an open laptop screen
isFeatured: false
tags: 
  - DevOps
  - Containers
  - Cloud Orchestration
  - Kubernetes
---

Kubernetes is a complex platform typically deployed across a number of servers to achieve high availability. However, creating multi-node clusters is often not practical for individual DevOps engineers performing local testing.

Fortunately, there are many options available for installing development Kubernetes clusters on a single machine. This allows DevOps engineers to verify many aspects of their code and deployment process before deploying to a shared cluster.

In this post, I look at some of the options available to DevOps engineers for running a development Kubernetes cluster.

## minikube

[minikube](https://minikube.sigs.k8s.io/docs/) is a cross-platform tool for creating single node Kubernetes clusters. It also provides [many examples showing how it can be configured with Continuous Integration (CI) servers](https://github.com/minikube-ci/examples), allowing ephemeral test clusters to be created and destroyed as part of automated tests.

Learn how to install minikube on Windows and connect it to Octopus in the post [Installing minikube on Windows](https://octopus.com/blog/minikube-on-windows).

## kind

[kind](https://kind.sigs.k8s.io/), which stands for Kubernetes in Docker, is a cross-platform tool supporting the creation of multi-node Kubernetes clusters all hosted as Docker containers. It can take a moment to wrap your mind around the concept of Docker running Kubernetes to orchestrate Docker containers, but the tool works well and has the advantage over other options of not requiring dedicated virtual machines in Windows and macOS.

You can find more information on integrating kind clusters with Octopus in the post [Kubernetes testing with kind](https://octopus.com/blog/getting-started-with-kind-and-octopus).

## MicroK8s

[MicroK8s](https://microk8s.io/) is both an option for local development on Windows, Linux, and macOS, and for production environments with the option of enterprise support and security maintenance. MicroK8s supports [addons](https://microk8s.io/docs/addons), providing a convenient way to install common features like an image registry, dashboards, ingress controllers, and more.

## K3s

[K3s](https://k3s.io/) is a lightweight, production-ready [Kubernetes distribution for Linux](https://rancher.com/docs/k3s/latest/en/installation/installation-requirements/#operating-systems) created by Rancher. K3s can create multi-node clusters on a single machine with [k3d](https://github.com/k3d-io/k3d), providing a convenient solution for creating a development cluster.

Learn how to integrate Rancher and Octopus in the post [Deploying to Rancher with Octopus Deploy](https://octopus.com/blog/deploy-to-rancher-with-octopus).

## k0s

[k0s](https://k0sproject.io/) is another lightweight, production-ready [Kubernetes distribution for Linux](https://docs.k0sproject.io/v1.23.6+k0s.2/system-requirements/#host-operating-system), with [experimental support for Windows server](https://docs.k0sproject.io/v1.23.6+k0s.2/experimental-windows/). It has built-in support for [creating a cluster on top of Docker](https://docs.k0sproject.io/v1.23.6+k0s.2/k0s-in-docker/#run-k0s-in-docker) for local development on Windows, Linux, and macOS.

## Docker Desktop

[Docker Desktop](https://docs.docker.com/desktop/kubernetes/) includes a standalone Kubernetes server and client for Windows, Linux, and macOS. Unlike the other options listed in this post, Docker Desktop is [commercial software](https://docs.docker.com/subscription/), requiring payment under some circumstances. 

Given Docker Desktop is the supported method for enabling the Docker engine in Windows, many developers may already have the tools required to spin up a development Kubernetes cluster, making it a convenient choice.

## Conclusion

Thanks to the eager adoption of Kubernetes by the open-source community, DevOps engineers have a wide range of options for quickly spinning up development clusters on their local machines and creating ephemeral clusters in CI tests. While Linux users have the most options, there are still plenty of active and well maintained projects for Windows and macOS users. So, with usually no more than one or two commands and a few minutes, DevOps engineers can spin up a local cluster ready for testing.

## Learn more

If you're looking to build and deploy containerized applications to AWS platforms such as EKS and ECS, check out the [Octopus Workflow Builder](https://octopusworkflowbuilder.octopus.com/#/). The Builder populates a GitHub repository with a sample application built with GitHub Actions workflows and configures a hosted Octopus instance with sample deployment projects demonstrating best practices such as vulnerability scanning and Infrastructure as Code (IaC). 

Happy deployments! 