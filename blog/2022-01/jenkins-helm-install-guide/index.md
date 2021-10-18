---
title: Helm Jenkins Installation
description: Learn how to install Jenkins via Helm
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

Kubernetes has grown in popularity to become one of the most popular platforms for hosting Docker containers. Kubernetes offers advanced orchestration features, networking capabilities, high availability, and volume management, and a wide ecosystem of supporting tools.

One of those supporting tools is Helm, which provides package management functionality for Kubernetes. Applications deployed by Helm are captured as charts, and Jenkins [provides a Helm chart](https://github.com/jenkinsci/helm-charts/blob/main/charts/jenkins/README.md) to deploy a Jenkins instance to Kubernetes.

In this post you'll learn how to install a Jenkins instance with Helm to Kubernetes, and connect a number of agents to perform build tasks.