---
title: From Continuous Delivery to Continuous Operations
description: In this post we create example runbooks to implement Continuous Operations
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

In the last blog post we integrated Jenkins and Octopus to trigger a deployment to Kubernetes once the Docker image was pushed to Docker Hub. We also added some additional environments in Octopus to represent a traditional Dev -> Test -> Prod progression. This left us with a traditional CI/CD pipeline with automated (if not necessarily automatic) deployments to our environments.

While a traditional CI/CD pipeline ends with a deployment to production, Octopus treats deployments as the beginning of a new phase called Continuous Operations (CO). By automating common tasks like database backups, log collection, and service restarts via runbooks, Octopus provides a complete CI/CD/CO pipeline covering the entire lifecycle of an application.

## 