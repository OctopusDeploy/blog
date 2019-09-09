---
title: A look at the Istio ServiceEntry
description: In this post we'll expose external URLs to Istio with the ServiceEntry resource.
author: matthew.casperson@octopus.com
visibility: private
published: 2020-01-01
metaImage:
bannerImage:
tags:
 - Octopus
---

In order to be able to make a network request, the endpoint being requested must be part of the Istio service registry. By default any Service in a Kubernetes is part of the service registry, but external URLs are not. To expose external network applications to Istio we use the [ServiceEntry](https://istio.io/docs/reference/config/networking/v1alpha3/service-entry/).

In this post we'll add a ServiceEntry resource to the Kubernetes cluster in order to direct our `proxy` application to some external resources.

## Redirecting internal requests to external resources

To demonstrate the ServiceEntry resource we'll direct the requests to `http://webserver` from the `proxy` to https://raw.githubusercontent.com/mcasperson/NodejsProxy/master/externalservice1.txt. This is a plain text file hosted by GitHub, which we are using to simulate an external API endpoint.

The first step is to expose the host `raw.githubusercontent.com` to the Istio service registry, which is achieved with a ServiceEntry resource.

In the YAML below we expose the host `raw.githubusercontent.com`, identified as being external to the Istio mesh via the `MESH_EXTERNAL` location, accepting HTTPS traffic on the standard port of 443, and resolved to an IP address using the DNS server available inside the Kubernetes cluster.

```Yaml
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: github
spec:
  hosts:
  - "raw.githubusercontent.com"
  location: MESH_EXTERNAL
  ports:
  - number: 443
    name: https
    protocol: TLS
  resolution: DNS
```

With the external service now part of the Istio service registry, we need to direct traffic to it. This is achieved with the VirtualService resource.

As before we are matching requests to the `webserver` host, and using the rewriting functionality to redirect the request to another location. However, unlike the pervious blogs, we are redirecting the traffic to an external location.

:::hint
It is important to set the `authority` to the same value as the `destination` `host`, because a large number of external services expect the host field in the HTTP request to be defined correctly. Failure to set the `authority` field can lead to 404 errors.
:::

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
        regex: ".*"
    rewrite:
      uri: "/mcasperson/NodejsProxy/master/externalservice1.txt"
      authority: raw.githubusercontent.com
    route:
    - destination:
        host: raw.githubusercontent.com
        port:
          number: 443
```

Because we have redirected a HTTP call to a HTTPS external location, we need to configure a DestinationRule to act as a TLS client. In the YAML below we have configured the DestinationRule to `SIMPLE`, which indicates that requests to the host should be conducted over a TLS connection.

```YAML
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: github
spec:
  host: "raw.githubusercontent.com"
  trafficPolicy:
    tls:
      mode: SIMPLE
```
