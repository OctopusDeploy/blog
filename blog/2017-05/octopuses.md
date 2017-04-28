---
title: "Octopuses"
description: "There are some compelling reasons to use multiple Octopus Server instances, but managing multiple instances is problemmatic. We want to make managing multiple related Octopus Server instances a first-class citizen of the Octopus world." 
author: michael.richardson@octopus.com
visibility: private
---

When we first built Octopus, we imagined it would be used by small teams to deploy applications to a dozen or so machines. Over time, we've [seen customers scale Octopus up to many thousands of machines](https://octopus.com/blog/octostats), deploying hundreds of different projects. At that scale, customers need their Octopus instances to be online at all times, so we support running a single [Octopus server across a multi-node, high availability cluster](https://octopus.com/high-availability).

One great big Octopus server isn't always a great idea though, especially when it's used by a large number of teams that don't share a lot in common. That was the case at Accenture, who [standardized on Octopus](https://channel9.msdn.com/Shows/ANZMVP/Updating-Octopus-Deploy-at-Accenture-with-Jim-Szubryt-and-Damian-Brady) across the organization, and had many hundreds of teams on a handful of very large Octopus servers. For their scenario, it made much more sense to split the big Octopus servers into lots of small ones, effectively giving each team or handful of teams their own small, isolated Octopus servers.

Another scenario where it makes sense to split into multiple Octopus Servers is for security isolation. Customers who [deploy their applications into PCI Compliant environments](https://octopus.com/docs/reference/pci-compliance-and-octopus-deploy) typically end up managing two separate Octopus Servers: one in their development/insecure environment, and one in their production/secure environment. In this scenario the difficulty is in synchronizing projects across the two security zones.

The more we thought about it, the more we realised there are quite a few compelling reasons why you may want to split into multiple instances of Octopus Server.

1. **Independent teams:** Your organization has multiple teams that work independently. Currently Octopus has many entities that are shared between Projects (e.g. Lifecycles, Variable Sets, Step Templates, etc). Separate Octopus Servers ensure your peas and carrots stay on their own sides of the plate.

1. **Scale:** A single server has finite resources. While High Available allows you to scale work across multiple servers, there are many situations where having large numbers of entities (Environments, Machines, Projects, etc) impacts performance and usability.

1. **Distributed Environments:** Similarly to 3, many organisations deploy to environments in multiple geographic regions.  Deployment times (particularly package transfers) can be dramatically reduced by hosting an Octopus instance in each location.

1. **Security:** For security (for example PCI DSS compliance) your organization doesn't allow network communication between development and production environments. Many customers address this by having an Octopus Server in each zone.

Now, based on some of these reasons, you go ahead and split your single Octopus Server instances, only to realise just how difficult it can be to manage them all.


You can work around all these difficulties using Octopus today, but there are a lot of trade offs.

- Offline drops (no output variables nor deployment logs or dashboard updates)

We want to make all of this easier, as first-class citizens of the Octopus world.

Here is an example of the architecture we want to support.

![Spaces and Zones example architecture](spaces-and-zones-architecture.png)