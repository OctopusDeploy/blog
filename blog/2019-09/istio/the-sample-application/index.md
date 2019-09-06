---
title: Exploring Istio - The Sample Application
description: In this post we look at a very simple sample application that we'll use to explore the functionality of Istio.
author: matthew.casperson@octopus.com
visibility: private
published: 2020-01-01
metaImage:
bannerImage:
tags:
 - Octopus
---

Istio is one of the most popular and powerful service meshes available for Kubernetes today. To understand the features it provides, it is useful to have a very simple sample application to make network requests that we can manipulate and configure via Istio.

The canonical example provided by the Istio project is [Bookinfo](https://istio.io/docs/examples/bookinfo/). Bookinfo is a small polyglot microservice application whose output can be tweaked by modifying network policies.

However I found Bookinfo too high level to really understand the features of Istio, and so instead we'll present a very simple Kubernetes deployment with a Node.js application proxying the request to various web servers, both internal and external to the cluster. By keeping things very simple, it is easy to see exactly what Istio is doing as network requests are being made.

## The proxy application

The public facing application in our example is a very simple Node.js application. This application makes a second network request to the service it is proxying, wraps up the response and returns it along with the time it took to make the request.

We'll use this proxy frontend to observe how network requests are routed across the network, to display any failed network requests, and to measure how long the requests took.

```javascript
code goes Here
```

## The external services

Inside this same repository we have a two text files [here](https://raw.githubusercontent.com/mcasperson/NodejsProxy/master/externalservice1.txt) and [here](https://raw.githubusercontent.com/mcasperson/NodejsProxy/master/externalservice2.txt). These will serve as mock external services for our proxy to consume thanks to the fact that GitHub allows you to view the raw contents of a file in a hosted repository. This means we don't have to go to the trouble of deploying a public service to return known values.

## The internal webservers

For the internal web servers we will run a second Node.js application that returns some static text, `Webserver V1` and `Webserver V2` in our case, along with the hostname of the container that is running the image. We will spin up 3 instances for each version, which means that we will have 6 instances running 2 versions of the server.

The different versions of the webserver will be labeled with `version: v1` or `version: v2`. This configuration will provide us with the opportunity to route and manage network traffic in interesting ways when we start looking at Istio's VirtualService and DestinationRule resources.

## An architecture diagram

Here is a top level overview of the sample application, using the [Kubernetes Deployment Language](https://blog.openshift.com/kdl-notation-kubernetes-app-deploy/) (KDL).

We have a load balancer service directing traffic to the pod created by the `proxy` deployment, which in turn requests the content from the two pods created by the deployments `webserverv1` and `webserverv2`. The web server content is then returned back to the browser.

Meanwhile there are two additional cluster IP services called `webserverv1` and `webserverv2` that aren't currently accessed. These have been created in preparation for Istio policies that will direct traffic in a more fine grained manner than we have established with this initial implementation.

![](istio-sample.svg "width=500")

When we open the application, we'll see the proxy wrapping up the response from either `webserverv1` or `webserverv2` (because the service points to both pods, and so will contact either one for any given request). We can also see the time it took to retrieve the proxied value.

![](output.png "width=500")

## Conclusion

The example application shown here is trivial and does not attempt to replicate any real world scenario. However it is well suited as a starting point from which we can add new networking functionality with Istio.

In the next post we'll introduce the Istio VirtualService resource.
