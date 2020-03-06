---
title: A first look at Tekton Pipelines
description: This blog explores Tekton Pipelines and discusses how they fit into the CI/CD ecosystem
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage:
bannerImage:
tags:
 - Octopus
---

Kubernetes is quickly evolving from a Docker orchestration platform to a general purpose cloud operating system. With [operators](https://octopus.com/blog/operators-with-kotlin) Kubernetes gains the ability to natively manage high level concepts and business processes, meaning you are no longer managing the building blocks of Pods, Services and Deployments, but instead describe the things those building blocks can create like web servers, databases, continuous deployments, certificate management and more.

When deployed to a Kubernetes cluster, Tekton Pipelines expose the ability to define and execute build tasks, inputs and outputs in the form of simple values or complex objects like Docker images, and to combine these resources all together in pipelines. These new Kubernetes resources and the controllers that manage them result in a headless CI/CD platform hosted by a Kubernetes cluster.

In this post we'll take a look at a simple build pipeline running on MicroK8S.

## Preparing the test cluster

For this post I'm using MicroK8S to provide the Kubernetes cluster.
