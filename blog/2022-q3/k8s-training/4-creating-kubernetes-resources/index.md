---
title: Creating Kubernetes pods, ReplicaSets, and deployments
description: Learn how to create Kubernetes pods, ReplicaSets, and deployments from YAML manifests, as part of our Kubernetes training series.
author: matthew.casperson@octopus.com
visibility: public
published: 2022-01-01-1200
metaImage: blogimage-kubernetes.png
bannerImage: blogimage-kubernetes.png
bannerImageAlt: An octopus on a sail boat
isFeatured: false
tags: 
  - DevOps
  - Containers
  - Cloud Orchestration
  - Docker 
---

This post is the 4th in our Kubernetes training series, providing DevOps engineers with an introduction to Docker, Kubernetes, and Octopus.

This video demonstrates Kubernetes pods, ReplicaSets, and deployments, deploying examples of each.

<p style="text-align:center"><iframe src="https://fast.wistia.net/embed/iframe/90jqp8rihi?videoFoam=true" title="Section 4 Video" allow="autoplay; fullscreen" allowtransparency="true" frameborder="0" scrolling="no" class="wistia_embed" name="wistia_embed" msallowfullscreen width="640px" height="360px"></iframe></p>

You can work through the series using the links below.

!include <k8s-training-toc>

## Resources

- [Octopus trial](https://octopus.com/start)

### Sample Pod YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: underwater
spec:
  containers:
  - name: webapp
    image: octopussamples/underwater-app
    ports:
    - containerPort: 80
```

### Sample ReplicaSet YAML

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: webapp
  template:
    metadata:
      labels:
        tier: webapp
    spec:
      containers:
      - name: webapp
        image: octopussamples/underwater-app
        ports:
        - containerPort: 80
```

### Sample Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: webapp
  template:
    metadata:
      labels:
        tier: webapp
    spec:
      containers:
      - name: webapp
        image: octopussamples/underwater-app
        ports:
        - containerPort: 80
```
  

## Learn more

If you're looking to build and deploy containerized applications to AWS platforms such as EKS and ECS, the [Octopus Workflow Builder](https://octopusworkflowbuilder.octopus.com/#/) populates a GitHub repository with a sample application built with GitHub Actions workflows and configures a hosted Octopus instance with sample deployment projects demonstrating best practices such as vulnerability scanning and Infrastructure as Code (IaC). 

Happy deployments! 