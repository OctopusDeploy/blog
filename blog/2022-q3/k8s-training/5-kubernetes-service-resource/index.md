---
title: Creating Kubernetes Services
description: Learn how to expose Pods to network traffic via a Service
author: matthew.casperson@octopus.com
visibility: private
published: 2022-01-01-1200
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

<iframe src="https://fast.wistia.net/embed/iframe/s3txr5gd65?videoFoam=true" title="Section5 Video" allow="autoplay; fullscreen" allowtransparency="true" frameborder="0" scrolling="no" class="wistia_embed" name="wistia_embed" msallowfullscreen width="100%" height="100%"></iframe>

!include <k8s-training-toc>

This video demonstrates how to expose Pods to network traffic via a Service.

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

## Links

* [kind load balancer](https://oc.to/ilYOx0)

## Learn more

If you are looking to build and deploy containerized applications to AWS platforms such as EKS and ECS, the [Octopus Workflow Builder](https://octopusworkflowbuilder.octopus.com/#/) populates a GitHub repository with a sample application built with GitHub Actions workflows and configures an Hosted Octopus instance with sample deployment projects demonstrating best practices such as vulnerability scanning and Infrastructure as Code (IaC). 

Happy deployments! 