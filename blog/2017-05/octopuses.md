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

2. **Scale:** A single server has finite resources. While High Available allows you to scale work across multiple servers, there are many situations where having large numbers of entities (Environments, Machines, Projects, etc) impacts performance and usability.    

3. **Geography:** 
    - 3a. **Teams:** Many organizations have teams that are geographically dispersed. Octopus Servers located in each region can be used to address network latency. 

     - 3b. **Environments:** Similarly to 3a, many organisations deploy to environments in multiple georgraphic regions.  Deployment times (particularly package transfers) can be dramatically reduced by hosting an Octopus instance in each location. 

4. **Security:** For security (for example PCI DSS compliance) your organization doesn't allow network communication between development and production environments. Many customers address this by having an Octopus Server in each zone. 


A primary focus for Octopus 4.0 will be making it easier to manage multiple Octopus instances.

To do this, we are planning to introduce two new concepts: _Spaces_ and _Zones_. 

We are going to talk about these in _much_ more detail in coming posts (you can expect RFC's on each in the next couple of weeks). 

The two concepts are orthogonal, but complementary. As a (very) brief introduction, Spaces are designed to address points **1**, **2**, and **3a** above, while Zones address points **3b** and **4**.

Both are essentially just an Octopus Server instance. The difference is in how they interact. 

Spaces are an isolated set of related entities (Projects, LifeCycles, Variable Sets, etc), managed by a central component to allow for authentication and easy navigation between Spaces. 

Zones will effectively allow you to split a Project across multiple Octopus instances, defining some Environments, Variables, Tenants etc on one instance, some on another. Releases will be able to promoted across Zones (manually if there is no network access). 

As mentioned, both will be covered in much more detail soon. 

In the meantime, below is an example of the architecture we want to support.

![Spaces and Zones example architecture](spaces-and-zones-architecture.png)