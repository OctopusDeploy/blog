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

In Octopus Deploy deployment targets communicate with the Octopus Cloud server using tentacles. We have introduced a new capability to configure polling tentacles to use port 443 as opposed to port 10943, which was the only previous option. Using port 443 typically complies with an organisation’s firewall rules and so avoids the need to get authorisation for a firewall rule exception. Port 10943 is an unassigned port in the [IANA port listing](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml?&page=120), and therefore looks suspicious to some. Approval of firewall exceptions can take 6 months and significant effort.

## Using port 443 for polling tentacles - How it works

The Octopus server requires polling tentacle traffic and the Web app/API traffic to be on separate ports. So that polling tentacles can also use port 443, we have configured a second url that is used to re-route polling tentacle traffic to a separate server port. The result is customers’ firewall rules only need to have port 443 open for outgoing traffic and there is no longer any need to have an exception approved to open port 10943.

For Octopus Cloud, this second URL is your Octopus Cloud URL with the word “polling” added as a sub domain i.e., https://polling.YOUR_INSTANCE.octopus.app. You, the customer, will need to configure the polling tentacle to use this additional url and port 443, nothing else to do for Octopus Cloud.

This is illustrated in the diagram.
Use the following (minus the backtics) to include images:

```
![Alt text, a description of the image](/path/to/image.png "width=500")*Optional caption text*
```
If including images, please include alt text. Alt text is primarily used to describe images to people unable to see them, and can be 125 characters max including spaces. You can also include an image caption if the reader would benefit from additional information or context.


Within Octopus Cloud we have configured a reverse proxy that allows web app traffic to pass through on 443 but redirects polling traffic to the Octopus server port 10943, which is what the Octopus server expects. This also makes this change backward compatible with previous polling tentacles that are still communicating over 10943.

### Some things to note

This capability is available from polling tentacle version 6.3.417 (or later), so you will need to upgrade any existing polling tentacles to use port 443. Also, because of the backward compatibility of this solution, it is possible to run a mix of polling tentacles, some configured to run over port 443 and others over 10943. This may be useful if you are in the process of updating polling tentacles to use port 443, but you have port 10943 polling tentacles that need to operate in the meantime.

### What about self-hosted Octopus Deploy?

While this development and blog is aimed at Octopus Cloud, it raises the question for self-hosted OD (Octopus Deploy) servers. For the cases where you have a single OD server, or a single OD execution server node and one or more UI nodes, then an analogous solution can be used. You, the customer, will need to configure the additional URL and implement the reverse proxy redirect of the polling tentacle traffic to the (single) OD execution server.

For multiple OD execution servers, a solution to use port 443 for polling tentacles traffic is possible but requires a more complex implementation that is beyond the scope of this blog. Please reach out to your Octopus Deploy contact or support if this is your need.

## Conclusion

Polling tentacles can be configured to use port 443 for tentacle communication in Octopus Cloud. This removes the need to have a non-standard port open in the customer firewall, so saving what can be significant effort and up to 6 months delay in getting policy exemptions and custom firewall rules approved.

A similar approach can be used for self-hosted OD where there is a single execution server. For multiple executions servers, while it is possible, configuration is more complex.


## Conclusion

Close off the post by restating the main points of the post, share any closing thoughts, and invite feedback.

## Learn more

- [Polling tentacles over port 443](https://octopus.com/docs/infrastructure/deployment-targets/tentacle/polling-tentacles-over-port-443)


## Watch the webinar: {webinar title here}

<iframe width="560" height="315" src="https://www.youtube.com/embed/F_V7r80aDbo" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

We host webinars regularly. See the [webinars page](https://octopus.com/events) for details about upcoming events, and live stream recordings.

Happy deployments!
