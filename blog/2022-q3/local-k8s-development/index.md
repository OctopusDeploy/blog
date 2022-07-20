---
title: Testing Kubernetes locally
description: Learn the tools available for developers testing Kubernetes on their local machines. 
author: name.surname@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: placeholderimg.png
bannerImage: placeholderimg.png
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - Containers
  - Cloud Orchestration
---

Kubernetes is a complex platform typically deployed across a number of servers to achieve high availability. However, creating multi-node clusters is often not practical for individual DevOps engineers performing local testing.

Fortunately, there are many options available for installing development Kubernetes clusters on a single machine. This allows DevOps engineers to verify many aspects of their code and deployment process before deploying their applications to a shared cluster.

In this post, we'll look at some of the options available to DevOps engineers for running a development Kubernetes cluster.

## minikube

[minikube](https://minikube.sigs.k8s.io/docs/) is a cross-platform tool for creating single node Kubernetes clusters. [It also provides many examples of how it can be configured with Continuous Integration (CI) servers](https://github.com/minikube-ci/examples), allowing ephemeral test clusters to be created and destroyed as part of automated tests.

Learn how to install minikube on Windows and connect it to Octopus in the post [Installing Minikube on Windows](https://octopus.com/blog/minikube-on-windows).

## kind

[kind](https://kind.sigs.k8s.io/), which stands for Kubernetes in Docker, is a cross-platform tool supporting the creation multi-node Kubernetes clusters all hosted as Docker containers. It can take a moment to wrap your mind around the concept of Docker running Kubernetes to orchestrate Docker containers, but the tool works well and has the advantage over other options of not requiring dedicated virtual machines in Windows and macOS.

You can find more information on integrating kind cluster with Octopus in the post [Kubernetes testing with KIND](https://octopus.com/blog/getting-started-with-kind-and-octopus).

## MicroK8s

[MicroK8s](https://microk8s.io/) is both an option for local development on Windows, Linux, and macOS, and for production environments with the option of enterprise support and security maintenance. MicroK8s supports [addons](https://microk8s.io/docs/addons) providing a convenient way to install common features like an image registry, dashboards, ingress controllers and more.

## K3s

[K3s](https://k3s.io/) is a lightweight, production ready [Kubernetes distribution for Linux](https://rancher.com/docs/k3s/latest/en/installation/installation-requirements/#operating-systems). K3s has the ability to create multi-node clusters on a single machine with [k3d](https://github.com/k3d-io/k3d), providing a convenient solution for creating a development cluster.

## k0s

[k0s](https://k0sproject.io/) is another lightweight, production ready [Kubernetes distribution for Linux](https://docs.k0sproject.io/v1.23.6+k0s.2/system-requirements/#host-operating-system), with [experimental support for Windows server](https://docs.k0sproject.io/v1.23.6+k0s.2/experimental-windows/). It has native support for [creating a cluster on top of Docker](https://docs.k0sproject.io/v1.23.6+k0s.2/k0s-in-docker/#run-k0s-in-docker) for local development on Windows, Linux, and macOS.

## Docker Desktop

[Docker Desktop](https://docs.docker.com/desktop/kubernetes/) includes a standalone Kubernetes server and client for Windows, Linux, and macOS. Unlike the other options listed in this post, Docker Desktop is [commercial software](https://docs.docker.com/subscription/), requiring payment under some circumstances. Given Docker Desktop is the supported method for enabling the Docker engine in Windows, many developers may find they already have the tools required to spin up a development Kubernetes cluster, making it a convenient choice.

## Conclusion

Thanks to the eagre adoption of Kubernetes by the open-source community, DevOps engineers have a wide range of options for quickly spinning up development clusters on their local machines and creating ephemeral clusters in CI tests. While Linux users have the most options, there are still plenty of active and well maintained projects for Windows and macOS users. So, with usually no more than one or two commands and a few minutes, DevOps engineers can spin up a local cluster ready for testing.

## Learn more

If you are looking to build and deploy containerized applications to AWS platforms such as EKS and ECS, the [Octopus Workflow Builder](https://octopusworkflowbuilder.octopus.com/#/) populates a GitHub repository with a sample application built with GitHub Actions workflows and configures an Hosted Octopus instance with sample deployment projects demonstrating best practices such as a vulnerability scanning and Infrastructure as Code (IaC). 

Happy deployments! 
