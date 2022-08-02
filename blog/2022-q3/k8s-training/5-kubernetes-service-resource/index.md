---
title: Creating Kubernetes services
description: Learn how to expose pods to network traffic via a service.
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

This post is the fifth in our Kubernetes training series, providing DevOps engineers with an introduction to Docker, Kubernetes, and Octopus. You’ll learn how to create an automated, multi-environment deployment process so you can deploy containerized applications with speed and reliability. 

This video demonstrates how to expose pods to network traffic via a service.

<p style="text-align:center"><iframe src="https://fast.wistia.net/embed/iframe/s3txr5gd65?videoFoam=true" title="Section5 Video" allow="autoplay; fullscreen" allowtransparency="true" frameborder="0" scrolling="no" class="wistia_embed" name="wistia_embed" msallowfullscreen width="640px" height="360px"></iframe></p>

## Sample Service YAML

```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp
spec:
  selector:
    app: webapp
  type: LoadBalancer
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

Work through the series using the links below.

!include <k8s-training-toc>

## Links

* [Octopus trial](https://octopus.com/start)
* [kind load balancer](https://oc.to/ilYOx0)

## Learn more

If you're looking to build and deploy containerized applications to AWS platforms such as EKS and ECS, the [Octopus Workflow Builder](https://octopusworkflowbuilder.octopus.com/#/) populates a GitHub repository with a sample application built with GitHub Actions workflows and configures a hosted Octopus instance with sample deployment projects demonstrating best practices such as vulnerability scanning and Infrastructure as Code (IaC). 

Happy deployments! 
