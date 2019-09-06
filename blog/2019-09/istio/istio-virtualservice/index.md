---
title: A look at the Istio VirtualService
description: In this blog post we add a VirtualService resource to the cluster to perform network routing
author: matthew.casperson@octopus.com
visibility: private
published: 2020-01-01
metaImage:
bannerImage:
tags:
 - Octopus
---

In the previous blog post we deployed a few simple web applications into a Kubernetes cluster as Deployments, and linked everything up with standard Services.

Because the webserver Service matches the labels for Pods created by both the `webserverv1` and `webserverv2` Deployments, calling the proxy will return the content any one of the webserver pods. This means we see the proxy returning the static content `WebServer V1` or `WebServer V2` with any given page refresh.

In this post we'll look at what a VirtualService is, how it relates to a standard Ingress, and add a VirtualService to the cluster to route and modify the requests made by the `proxy` Pod to the `webserver` Service.

## An overview of the VirtualService resource

A VirtualService is a Custom Resource Definition (CRD) provided by Istio. A VirtualService acts in much the same capacity as a traditional Kubernetes Ingress, in that a VirtualService matches traffic and directs it to a Service.

However, a VirtualService can be much more specific in the traffic it matches and where that traffic is sent, and offers a lot of additional functionality to manipulate the traffic.

An Ingress can match incoming traffic based on the HTTP host and path. Depending on the Ingress Controller that is installed in the cluster, the path can match wildcard values or even accept regular expressions. But such advanced features are not universal among Ingress Controllers; for example, the [Nginx Ingress Controller supports regex paths](https://kubernetes.github.io/ingress-nginx/user-guide/ingress-path-matching/), while the Google Kubernetes Engine Ingress Controller only supports very [selective uses of the asterisk as a wildcard](https://cloud.google.com/kubernetes-engine/docs/concepts/ingress#multiple_backend_services).

A VirtualService on the other hand can match traffic based on HTTP host, path (with full regular expression support), method, headers, ports, query parameters and more.

Once an Ingress has matched the incoming traffic, it is then directed to a Service. This is true of a VirtualService, however when combined with a RouteDestination, a VirtualService can direct traffic to specific subsets of Pods referenced by a Service. We'll see this in the next blog post.

Perhaps the biggest difference between an Ingress and a VirtualService is that a VirtualService can intelligently manage the traffic it matches by allowing requests to be retried, injecting faults or delays for testing, and rewriting or redirecting requests. Implementing this functionality in the infrastructure removes the need for each individual application to implement and manage it themselves, providing for a much more consistent networking experience.

Now that we know what a VirtualService can do, lets add one to the network to see it in action.

## A minimal example

Let's start with one of the simplest examples of a VirtualService. The YAML for this example is shown below.

```YAML
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: webserver
spec:
  hosts:
  - "webserver"
  http:
  - route:
    - destination:
        host: webserverv1.default.svc.cluster.local
```

Let's break this file down.

We start with the hostname of a request that this VirtualService will match. Here we have matched any call to the `webserver` service, which if you recall from the architecture diagram is the service that our `proxy` application calls.

```YAML
hosts:
- "webserver"
```

We then create some rules to direct any HTTP traffic that matches the hostname by configuring the `http` property. Under that property we define the `route` which sets the `destination` to another Service.

```YAML
http:
- route:
  - destination:
      host: webserverv1.default.svc.cluster.local
```

When this VirtualService is created in the cluster, we will see that requests made by the `proxy` are now routed to the `webserverv1` Service instead of the original `webserver` Service. The end result is that our proxy will only ever request content from the Pods created by the `webserverv1` Deployment, meaning we will only ever see messages like `Proxying value: WebServer V1 from ...` when the `proxy` application is called.

## Injecting network faults

Injecting faults into requests is a great way to test how your applications respond to failed requests.

```YAML
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: webserver
spec:
  hosts:
  - "webserver"
  http:
  - route:
    - destination:
        host: webserverv1.default.svc.cluster.local
    fault:
      abort:
        percentage:
          value: 50
        httpStatus: 400          
```

The `fault` property has been configured to abort 50% of requests with a HTTP response code of 400.

```
fault:
  abort:
    percentage:
      value: 50
    httpStatus: 400
```

We can see these failed requests printed by the proxy as the request it makes to the next service is aborted by Istio.

![](faults.png "width=500")

## Injecting network delays



![](delay.png "width=500")
