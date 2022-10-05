---
title: The difference between ClusterIP, NodePort, and LoadBalancer Kubernetes services
description: Learn the differences between the three kinds of Kubernetes services
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Kubernetes
---

Kubernetes services provide three different types: `ClusterIP`, `NodePort`, and `LoadBalancer`. Knowing which type of service to configure is critical to allowing clients to establish network connections while not exposing services to unnecessary traffic.

In this post I'll discuss the three types of services and when they should be used.

## The ClusterIP service type

The YAML below defines a service of type `ClusterIP` that directs traffic on port 80 (defined by the `port` property) to port 8080 (defined by the `targetPort` property) on any pods with the label `app` set to `web` (defined by the `selector` property):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: ClusterIP
  selector:
    app: web
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

`ClusterIP` services expose pods to internal network traffic, but do not expose pods to any external traffic. For example, you may expose a database to other pods via a `ClusterIP` service because external clients should never have direct access to the database.

`ClusterIP` services expose the smallest surface area and should be used when pods only need to be exposed to other pods in the cluster.

The diagram below shows how pods within the same cluster can communicate via the `ClusterIP` service:

![ClusterIP diagram](clusterip.png "width=500")

## The NodePort service type

