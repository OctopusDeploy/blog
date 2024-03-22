---
title: Creating Kubernetes services
description: Learn how to expose pods to network traffic via a service, as part of our Kubernetes training series.
author: matthew.casperson@octopus.com
visibility: public
published: 2022-01-01-1200
metaImage: blogimage-testingkubernetes-2022.png
bannerImage: blogimage-testingkubernetes-2022.png
bannerImageAlt: Kubernetes logo on an open laptop screen
isFeatured: false
tags: 
  - DevOps
  - Containers
  - Cloud Orchestration
  - Continuous Delivery
  - Docker 
  - Kubernetes
  - Kubernetes Training
---

This post is the 5th in our Kubernetes training series, providing DevOps engineers with an introduction to Docker, Kubernetes, and Octopus. 

This video demonstrates how to expose pods to network traffic via a service.

[If you don't already have Octopus account, you can start a free trial.](https://oc.to/octopus-k8s-training-trial)

:::hint
MetalLB has been updated since this video was recorded to use Custom Resource Definitions (CRDs).

You can still apply these manifests to apply version 0.12, which is the version used by this video:

* https://raw.githubusercontent.com/metallb/metallb/v0.12.1/manifests/namespace.yaml
* https://raw.githubusercontent.com/metallb/metallb/v0.12.1/manifests/metallb.yaml

And then apply this ConfigMap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 172.19.255.200-172.19.255.250
```
:::

<p style="text-align:center"><iframe width="560" height="315" src="https://www.youtube.com/embed/lkA-NUsarGo?si=Fn4LBZu4xqVNOYCn" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe></p>

You can work through the series using the links below.

!include <k8s-training-toc>

## Resources

- [Octopus trial](https://oc.to/octopus-k8s-training-trial)
- [kind load balancer](https://oc.to/ilYOx0)

### Sample Service YAML

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


## Learn more

If you're looking to build and deploy containerized applications to AWS platforms such as EKS and ECS, the [Octopus Workflow Builder](https://octopusworkflowbuilder.octopus.com/#/) populates a GitHub repository with a sample application built with GitHub Actions workflows and configures a hosted Octopus instance with sample deployment projects demonstrating best practices such as vulnerability scanning and Infrastructure as Code (IaC). 

Happy deployments! 
