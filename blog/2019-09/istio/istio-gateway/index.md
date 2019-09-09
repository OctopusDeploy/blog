---
title: A look at the Istio Gateway resource
description: This post will expose an Istio Gateway resource to direct external traffic into the cluster
author: matthew.casperson@octopus.com
visibility: private
published: 2020-01-01
metaImage:
bannerImage:
tags:
 - Octopus
---

Up until this point our Kubernetes cluster has taken traffic from a standard load balancer Service resource, which thanks to the fact that our cluster is hosted by AWS, is exposed by an ELB with a public IP. External traffic hitting this load balancer is directed to our `proxy` application, and from here we have used Istio to route the internal traffic.

As well as routing internal traffic, Istio can also route external traffic entering the cluster. The Gateway resource is used by Istio to receive external traffic and route it as it enters the cluster.

In this post we'll add a Gateway resource to the cluster to replace the load balancer Service resource we have been relying on.

## Defining the Gateway resource

Here is a minimal example of a Gateway resource.

```YAML
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: default-gateway
  namespace: istio-system
spec:
  selector:
    istio: ingressgateway
  servers:
  - hosts:
    - '*'
    port:
      name: http
      number: 80
      protocol: HTTP
```

There are some important settings here to be discussed.

First, this Gateway resource has been created in the `istio-system` namespace.

```YAML
namespace: istio-system
```

This is because this Gateway resource is going to be bound to a load balancer Service resource created when Istio was installed. The Service resource is called `istio-ingressgateway`, and has a label of `istio: ingressgateway`.

![](ingressgateway.png "width=500")

*The load balancer service created by Istio during installation.*

We attach this Gateway resource to the `istio-ingressgateway` Service with the label selectors.

```YAML
selector:
  istio: ingressgateway
```

This Gateway resource will accept all HTTP traffic from any host.

```YAML
- hosts:
  - '*'
  port:
    name: http
    number: 80
    protocol: HTTP
```
