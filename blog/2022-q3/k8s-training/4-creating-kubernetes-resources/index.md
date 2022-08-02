---
title: Creating Kubernetes Services
description: Learn about Kubernetes Pods, ReplicaSets, and Deployments
author: matthew.casperson@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: blogimage-kubernetes.png
bannerImage: blogimage-kubernetes.png
bannerImageAlt: An octopus on a sail boat
isFeatured: false
tags: 
  - DevOps
  - Containers Series
  - Containers
  - Cloud Orchestration
---

<iframe src="https://fast.wistia.net/embed/iframe/90jqp8rihi?videoFoam=true" title="Section 4 Video" allow="autoplay; fullscreen" allowtransparency="true" frameborder="0" scrolling="no" class="wistia_embed" name="wistia_embed" msallowfullscreen width="100%" height="100%"></iframe>

This video demonstrates Kubernetes Pods, ReplicaSets, and Deployments, deploying examples of each.

!include <k8s-training-toc>

## Sample Pod YAML

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

## Sample ReplicaSet YAML

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

## Sample Deployment YAML

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

## Links

* [Octopus Trail](https://octopus.com/start)

## Learn more

If you are looking to build and deploy containerized applications to AWS platforms such as EKS and ECS, the [Octopus Workflow Builder](https://octopusworkflowbuilder.octopus.com/#/) populates a GitHub repository with a sample application built with GitHub Actions workflows and configures an Hosted Octopus instance with sample deployment projects demonstrating best practices such as vulnerability scanning and Infrastructure as Code (IaC). 

Happy deployments! 