---
title: Polling tentacles over standard ports
description: Introduces the capability to use port 443(https:) for polling tentacles instead of the non-standard 10943. 
author: jonathan.hardy@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - Product
  - Tentacles
---

In Octopus Deploy deployment targets communicate with the Octopus Cloud server using tentacles. Now polling tentacles can use port 443, instead of port 10943. Port 443 often meets an organisation’s firewall rules for outgoing encrypted traffic. This avoids the need for authorisation of a firewall rule exception. Port 10943 is an unassigned port in the [IANA port listing](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml?&page=120), and therefore looks suspicious to some. Approval of firewall exceptions can take 6 months and significant effort.

## Using port 443 for polling tentacles - how it works

The Octopus server needs polling tentacle and web app traffic to be on separate ports. So that polling tentacles can also use port 443, we have configured a second url.  A reverse proxy re-routes polling tentacle traffic to the expected server port. So, customers’ firewall rules only need to have port 443 open for outgoing traffic. This avoids the need to apply to security to grant an exception and open port 10943.

This second URL is your Octopus Cloud URL prefixed with “polling” e.g., https://polling.YOUR_INSTANCE.octopus.app. Using this new domain and port 443 when configuring the polling tentacles is all you need to do.

![an image shows polling tentacles configured to use a second url and port 443 so that traffic passes unhindered through the customer firewall. It then shows the traffic entering the Octopus Cloud firewall on 443 and being redirected to port 10943 on the Octopus server.](OC-polling-tentacles-over-443.png "width=500")*Figure 1: Octopus Cloud-solution overview*

In Octopus Cloud, a reverse proxy redirects polling traffic to port 10943 on the Octopus server. Web app traffic passes through on port 443, which is what the Octopus server expects. This is backward compatible with polling tentacles that are still communicating over 10943.

### Some things to note

This capability is available from polling tentacle version 6.3.417 (or later). You will need to upgrade any existing polling tentacles to use port 443. It is possible to run a mix of polling tentacles, some configured to run over port 443 and others over 10943. This may be useful if you are updating polling tentacles to use port 443 and existing 10943 tentacles still need to work.

### What about self-hosted Octopus Deploy?

This development and blog are for Octopus Cloud. What about self-hosted OD (Octopus Deploy) servers? This development works for a single OD server, or a single OD execution server node and one or more UI nodes. Also, you will need to add the second URL and a reverse proxy redirect for polling tentacle traffic.

A solution for many OD execution servers needs more complex configuration. This is beyond the scope of this blog. Please reach out to your Octopus Deploy contact or support if this is your need.

## Conclusion

Octopus Cloud can use port 443 for polling tentacle communication. This removes the need to have a non-standard port open in the customer firewall. This saves the effort and up to 6 months delay for approval of firewall exemptions and custom rules.

A similar approach works for self-hosted OD where there is a single execution server. For multiple executions servers, while it is possible, configuration is more complex.

## Learn more

- [Polling tentacles over port 443](https://octopus.com/docs/infrastructure/deployment-targets/tentacle/polling-tentacles-over-port-443)


## Watch the webinar: {Polling tentacles over standard ports}

<iframe width="560" height="315" src="https://youtu.be/a4yeAwWwXi8" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


We host webinars regularly. See the [webinars page](https://octopus.com/events) for details about upcoming events, and live stream recordings.

Happy deployments!
