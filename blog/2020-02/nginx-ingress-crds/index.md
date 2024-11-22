---
title: A look at the new NGINX VirtualServer and VirtualServerRoute resources
description: See the features of the new custom resource definitions introduced with the NGINX v1.5 ingress controller.
author: matthew.casperson@octopus.com
visibility: public
published: 2020-02-19
metaImage: nginx-ingress-crds.png
bannerImage: nginx-ingress-crds.png
bannerImageAlt: NGINX VirtualServer and VirtualServerRoute resources
tags:
 - DevOps
 - Kubernetes
---

![NGINX VirtualServer and VirtualServerRoute resources](nginx-ingress-crds.png)

Kubernetes `Ingress` resources provide a way of configuring incoming HTTP traffic and make it easy to expose multiple services through a single public IP address.

NGINX has long provided one of the most popular ingress controllers, but anything more than a proof of concept deployment inevitably meant customizing routing rules beyond the standard properties exposed by `Ingress` resources.

Until recently, the solution was to define these additional settings via annotations or providing configuration blocks in configmaps. But with version 1.5, the NGINX ingress controller provides two custom resource definitions (CRDs) defining more complex networking rules than the baseline `Ingress` resources.

In this post, we’ll explore some of the new functionality provided by the `VirtualServer` and `VirtualServerRoute` CRDs.

## The sample Kubernetes cluster

For this blog, I used the Kubernetes distribution bundled with Docker Desktop:

![](dockerdesktop.png "width=500")

 I then deployed the sample application created for the [Istio blog series](https://octopus.com/blog/istio/the-sample-application), which can be installed with:

```
kubectl apply -f https://raw.githubusercontent.com/mcasperson/NodejsProxy/master/kubernetes/example.yaml
```

The resulting cluster looks like this:

![](sampleapp.svg "width=500")

I used Helm to install NGINX, but it’s not as easy as it could be. The [GitHub docs](https://github.com/nginxinc/kubernetes-ingress/tree/master/deployments/helm-chart#installing-via-helm-repository) point you to the Helm repo, which failed for me. The chart from the official Helm repository at https://kubernetes-charts.storage.googleapis.com/ did not include the CRDs.

The solution was to clone the NGINX Git repo and install the Helm chart from a local file. These commands worked with Helm 3:

```
git clone https://github.com/nginxinc/kubernetes-ingress/
cd kubernetes-ingress/deployments/helm-chart
helm install nginx-release .
```

## A basic NGINX VirtualServer

We’ll start with a basic `VirtualServer` resource to expose the proxy.

```YAML
apiVersion: k8s.nginx.org/v1
kind: VirtualServer
metadata:
  name: virtualserver
spec:
  host: localhost
  upstreams:
  - name: proxy
    service: proxy
    port: 80
  routes:
  - path: /
    action:
      pass: proxy
```

The `upstreams` property defines the services that traffic can be sent to. In this example, we direct traffic to the `proxy` service.

The `routes` match incoming requests and perform an action in response. Typically the action is to direct traffic to an upstream server, which we have done with the `action.pass: proxy` configuration.

This `VirtualServer` replicates the functionality we might otherwise have defined in an `Ingress` resource, and once deployed to the cluster, we can open http://localhost/whatever/you/want to view the proxy web application.

## Custom actions

Things become interesting when we start digging into the new functionality the `VirtualServer` exposes.

Instead of passing requests to an upstream service, the `VirtualServer` can redirect the client to a new URL. Here we direct traffic back to the NGINX homepage:

```YAML
apiVersion: k8s.nginx.org/v1
kind: VirtualServer
metadata:
  name: virtualserver
spec:
  host: localhost
  routes:
  - path: /
    action:
      redirect:
        url: http://www.nginx.com
        code: 301
```

In this example, we define the content to be returned directly in the `VirtualServer` resource. This is great for testing:

```YAML
apiVersion: k8s.nginx.org/v1
kind: VirtualServer
metadata:
  name: virtualserver
spec:
  host: localhost
  routes:
  - path: /
    action:
      return:
        code: 200
        type: text/plain
        body: "Hello World\n"
```

## Traffic splitting

Traffic splitting can be used for canary deployments by directing a percentage of traffic to a new service. Here we configure the `VirtualServer` to pass traffic to the webserver services, splitting traffic between `webserverv1` and `webserverv2`.

```YAML
apiVersion: k8s.nginx.org/v1
kind: VirtualServer
metadata:
  name: virtualserver
spec:
  host: localhost
  upstreams:
  - name: webserverv1
    service: webserverv1
    port: 80
  - name: webserverv2
    service: webserverv2
    port: 80
  routes:
  - path: /
    splits:
    - weight: 80
      action:
        pass: webserverv1
    - weight: 20
      action:
        pass: webserverv2
```

## Load balancing

In [iptables proxy mode](https://kubernetes.io/docs/concepts/services-networking/service/#proxy-mode-iptables), endpoints available to a service are chosen at random. If you look at the diagram above, the `webserver` service directs traffic to both `webserverv1` and `webserverv2` deployments, so traffic to the `webserver` service would be randomly distributed between all pods.

NGINX allows us to specify the [load balancing rules](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/#choosing-a-load-balancing-method) used to direct traffic to upstream services. In the example below, we set the `lb-method` property to the `ip_hash` load balancing algorithm, ensuring a client is always sent to the same backend pod:

```YAML
apiVersion: k8s.nginx.org/v1
kind: VirtualServer
metadata:
  name: virtualserver
spec:
  host: localhost
  upstreams:
  - name: webserver
    service: webserver
    port: 80
    lb-method: ip_hash
  routes:
  - path: /
    action:
      pass: webserver
```

## Timeouts, retries, and keepalives

Lower level configuration details like connection timeouts, retries, and keepalives used to be defined as annotations on `Ingress` resources. With a `VirtualServer` resource, these settings are now exposed as first-class properties:

```YAML
apiVersion: k8s.nginx.org/v1
kind: VirtualServer
metadata:
  name: virtualserver
spec:
  host: localhost
  upstreams:
  - name: webserver
    service: webserver
    port: 80
    fail-timeout: 10s
    max-fails: 1
    max-conns: 32
    keepalive: 32
    connect-timeout: 30s
    read-timeout: 30s
    send-timeout: 30s
    next-upstream: "error timeout non_idempotent"
    next-upstream-timeout: 5s
    next-upstream-tries: 10
    client-max-body-size: 2m
  routes:
  - path: /
    action:
      pass: webserver
```

## Adding VirtualServerRoutes

`VirtualServer` resources can delegate the request to a `VirtualServerRoute`. This allows a master `VirtualServer` resource to configure top-level traffic rules with `VirtualServerRoute` resources handling more specific routing rules.

In the example below, requests are sent to a `VirtualServerRoute` resource which selects one of three possible upstream services based on the path:

```YAML
apiVersion: k8s.nginx.org/v1
kind: VirtualServer
metadata:
  name: virtualserver
spec:
  host: localhost
  routes:
  - path: /
    route: virtualserverroute
---
apiVersion: k8s.nginx.org/v1
kind: VirtualServerRoute
metadata:
  name: virtualserverroute
spec:
  host: localhost
  upstreams:
  - name: proxy
    service: proxy
    port: 80
  - name: webserverv1
    service: webserverv1
    port: 80
  - name: webserverv2
    service: webserverv2
    port: 80
  subroutes:
  - path: /webserverv1
    action:
      pass: webserverv1
  - path: /webserverv2
    action:
      pass: webserverv2
  - path: /
    action:
      pass: proxy
```

## Conclusion

Ingress controllers started as a cottage industry in the Kubernetes ecosystem, but as each provider differentiated itself from the competition with new configuration options and features, the annotations added to `Ingress` resources made them unwieldy and unportable.

By implementing CRDs, NGINX exposes advanced functionality with verifiable properties. I expect these CRDs will be enriched even further as additional common use cases are identified.
