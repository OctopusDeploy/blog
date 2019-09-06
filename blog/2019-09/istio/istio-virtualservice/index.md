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

In the previous blog post we deployed a few simple Node.js web applications into a Kubernetes cluster as Deployments, and linked everything up with standard Services.

The networking so far has only used standard Kubernetes resources to configure it. This works, but falls a little short when it comes to directing traffic between different versions of upstream APIs. You will have noticed that the `proxy` application is returning content from the Pods created by both the `webserverv1` and `webserverv2` Deployments, which is unlikely to be the desired result had this been a real world deployment.

In this post we'll look at what a VirtualService is, how it relates to a standard Ingress, and add a VirtualService to the cluster to route and modify the requests made by the `proxy` Pod to the `webserver` Service.

## An overview of the VirtualService resource

A VirtualService is a Custom Resource Definition (CRD) provided by Istio. A VirtualService acts in much the same capacity as a traditional Kubernetes Ingress, in that a VirtualService matches traffic and directs it to a Service.

However, a VirtualService can be much more specific in the traffic it matches and where that traffic is sent, and offers a lot of additional functionality to manipulate the traffic along the way.

For comparison, an Ingress can match incoming traffic based on the HTTP host and path. Depending on the Ingress Controller that is installed in the cluster, the path can match wildcard values or even accept regular expressions. But such advanced features are not universal among Ingress Controllers; for example, the [Nginx Ingress Controller supports regex paths](https://kubernetes.github.io/ingress-nginx/user-guide/ingress-path-matching/), while the Google Kubernetes Engine Ingress Controller only supports very [selective uses of the asterisk as a wildcard](https://cloud.google.com/kubernetes-engine/docs/concepts/ingress#multiple_backend_services).

A VirtualService on the other hand can match traffic based on HTTP host, path (with full regular expression support), method, headers, ports, query parameters and more.

Once an Ingress has matched the incoming traffic, it is then directed to a Service. This is also true of a VirtualService, however when combined with a RouteDestination, a VirtualService can direct traffic to specific subsets of Pods referenced by a Service. For example, you may want to direct traffic only to pods that have the `version: v2` label applied. We'll see this in the next blog post.

Perhaps the biggest difference between an Ingress and a VirtualService is that a VirtualService can intelligently manage the traffic it matches by allowing requests to be retried, injecting faults or delays for testing, and rewriting or redirecting requests. Implementing this functionality in the infrastructure layer removes the need for each individual application to implement and manage it themselves, providing for a much more consistent networking experience.

Now that we know what a VirtualService can do, lets add some to the network to see the effects that they have.

## A minimal example

Let's start with one of the simplest examples of a VirtualService. The YAML for this example is shown below.

```YAML
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: webserver
spec:
  hosts:
  - webserver
  http:
  - route:
    - destination:
        host: webserverv1
```

Let's break this file down.

We start with the hostname of a request that this VirtualService will match. Here we have matched any call to the `webserver` Service, which if you recall from the architecture diagram is the Service that our `proxy` application calls.

```YAML
hosts:
- webserver
```

We then create some rules to direct any HTTP traffic that matches the hostname by configuring the `http` property. Under that property we define the `route` which sets the `destination` to another Service called `webserverv1`.

```YAML
http:
- route:
  - destination:
      host: webserverv1
```

When this VirtualService is created in the cluster, we will see that requests made by the `proxy` application are now routed to the `webserverv1` Service instead of the original `webserver` Service. The end result is that our proxy will only request content from the Pods created by the `webserverv1` Deployment, meaning we will only see messages like `Proxying value: WebServer V1 from ...` when the `proxy` application is called.

![](basic.png "width=500")

*This VirtualService directs all traffic to the webserverv1 Service.*

## Injecting network faults

Injecting faults into requests is a great way to test how your applications respond to failed requests. Here is the YAML of a VirtualService that has been configured to inject random network faults.

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
        host: webserverv1
    fault:
      abort:
        percentage:
          value: 50
        httpStatus: 400          
```

The `fault` property has been configured to abort 50% of requests with a HTTP response code of 400.

```YAML
fault:
  abort:
    percentage:
      value: 50
    httpStatus: 400
```

We can see these failed requests printed by the proxy as the request it makes to the next Service is aborted by Istio.

![](faults.png "width=500")

*50% of requests will now fail like this.*

## Injecting network delays

The response from a network call can be artificially delayed, giving you a chance to test how poor networking or unresponsive applications can affect your code. The VirtualService below has been configured to add random delays to network requests.

```YAML
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: webserver
spec:
  hosts:
  - webserver
  http:
  - route:
    - destination:
        host: webserverv1
    fault:
      delay:
        percentage:
          value: 50
        fixedDelay: 5s  
```

The `fault` property has been configured to add a delay of 5 seconds to 50% of network requests.

```YAML
fault:
  delay:
    percentage:
      value: 50
    fixedDelay: 5s  
```

We can see these delays in the timing information presented by the `proxy` application.

![](delay.png "width=500")

*50% of network calls from the proxy application will now take 5 seconds to complete.*

## Redirecting requests

HTTP requests can be redirected (i.e. by returning a HTTP 301 response code) to direct the client to a new location. The VirtualService below redirects requests made to the root path of one Service to a new path on a new Service.

```YAML
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: webserver
spec:
  hosts:
  - webserver
  http:
  - match:
    - uri:
        exact: "/"
    redirect:
      uri: /redirected
      authority: webserverv1
```

We have used an exact match to the root path to redirect the request to http://webserverv1/redirected.

```YAML
- match:
  - uri:
      exact: "/"
  redirect:
    uri: /redirected
    authority: webserverv1
```

The `proxy` shows the details of the redirected call.

![](redirect.png "width=500")

*The proxied application is now called on the path /redirected.*

## Rewriting requests

Rewriting requests is much like redirecting them, only the routing is all done server side and the client does not know that the request was changed to a new path. The VirtualService below rewrites requests made to the root path of one Service and routes them to a new path on a new Service.

```Yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: webserver
spec:
  hosts:
  - webserver
  http:
  - route:
    - destination:
        host: webserverv1  
    match:
    - uri:
        exact: "/"
    rewrite:
      uri: /rewritten
```

We have used an exact match to the root path to rewrite the request to http://webserverv1/rewritten.

```
match:
- uri:
    exact: "/"
rewrite:
  uri: /rewritten
```

![](rewritten.png "width=500")

*The proxied application is now called on the path /rewritten.*

## Retying requests

Retrying failed requests is a common strategy to dealing with network errors or unresponsive applications. The VirtualService below will retry failed requests.

```YAML
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: webserver
spec:
  hosts:
  - webserver
  http:
  - route:
    - destination:
        host: webserver
        subset: v1    
    retries:
      attempts: 3
      perTryTimeout: "2s"
      retryOn: "5xx"      
    timeout: "10s"         
```

Here we configure the VirtualService to retry any request that resulted in a 500 error code up to 3 times.

```YAML
retries:
  attempts: 3
  perTryTimeout: 2s
  retryOn: "5xx
```

The timeout was set to work around a [bug in Istio](https://github.com/kubernetes/ingress-gce/issues/181) that sets `perTryTimeout` to `0` if the `timeout` is not set.

```YAML
timeout: "10s"
```

We can see that requests that result in proxied requests to an endpoint that should fail 25% of the time only rarely respond with a 500 code, but the requests can take seconds as the retries are delayed.

![](retry.png "width=500")

*The /failssometimes path will return a 500 code 25% of the time, but with retries we rarely see a failure.*

## Conclusion

In this blog post we have seen the major features of the Istio VirtualService. The networking in our sample application has been redirected, retried, and artificially slowed down or failed all from the VirtualService, and without modifying the code from the original applications.

This shows the power of lifting this kind of networking responsibility from the application code to the infrastructure layer.

In the next post we'll see how to further customize the function of the network through the DestinationRule CRD.
