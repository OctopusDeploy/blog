---
title: Kubernetes Pod Service Accounts
description: TODO 
author: ray.nham@octopus.com
visibility: private
published: 2022-08-28-1400
tags:
 - DevOps
---

I'm happy to announce that we're introducing simpler authentication for Octopus Workers running as Containers in Kubernetes clusters. In this blog post, I'll introduce the new support and show you how to get started to take advantage of this update.

Octopus introduced Workers in version 20xx.x as a way to shift deployment work off the Octopus Server to pools of machines. Workers are useful for teams to benefit 1, benefit 2, benefit 3 (or whatever). It's also possible for teams to create a pool of workers as a containers running in a Kubernetes Cluster. Mention why this is valuable. Why is it better to run workers as containers in K8s. Then talk about the pain ... One of the challenges this introduces is authentication because ... . 

When running a worker as a container in a Kubernetes cluster, it is now possible to connect back to the parent cluster using the service account token and cluster certificate files mounted as files in the pod. This allows Kubernetes workers to manage the cluster they are deployed to without sending down any additional credentials.

## Add a Kubernetes Cluster as a deployment target

I thought we might want to have a super short intro to Kubernetes clusters in Octopus. Just briefly state where they are and how to configure them but link to docs or blog post for in-depth the details.

## Create a work pool with a worker as a container

Add a short intro to work pools and how to add a worker as a container. I don't actually know how to do this. Similar to the above point, I think a balance between adding some context to help readers is good but link to the docs for the full details.

## Configuring Worker Authentication 

Describe how to do this. 

## Conclusion

Summarise the article in one or two sentences and mention the benefits of this. :) 

Happy deployments!
