---
title: Octopus Enterprise Deployment Patterns
description: Learn the common deployment strategies enterprise teams can adopt with Octopus
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

Supporting software deployments and maintaining applications in large enterprise environments is often not as simple as configuring a single, shared Octopus instance that everyone can use. Practical constraints such as network latency between geographically distributed teams, the desire for business units to control their own infrastructure and processes, business acquisitions that bring established DevOps systems, and compliance with standards like PCI are common and valid reasons to configure multiple Octopus spaces and instances.

Over time we have seen a number of common patterns emerge with Octopus installations as enterprise environments address these concerns. In this post I'll break these patterns down and describe how each pattern can be used to address common scenarios in enterprise environments.

## Independent space per business unit/application

![Separate Spaces](separate_spaces.png width=500px)

The most common pattern is to partition a single Octopus installation into separate spaces. Octopus is fairly agnostic as to what any individual space represents, but it is common to provide a space for business units or application stacks. As long as the space represents a stable context for the projects it holds (meaning Octopus projects are unlikely to move between spaces even as people move between teams or security requirements change), spaces are a convenient method of splitting projects and defining security boundaries.

This pattern is very easy to implement as it often involves little more that creating a new space and assigning security permissions. We expect most Octopus users to naturally adopt spaces as their usage of the platform grows.

However, spaces do have some limitations. Because spaces are limited to a single Octopus installation, and Octopus installations require a low latency connection to the database, spaces do not provide the ability to co-locate Octopus with geographically dispersed teams. In addition, all tasks initiated by spaces used a shared task queue, commonly known as the "noisy neighbor" problem.

| Feature  | Solves  |
|---|---|
| Independent projects, runbooks, dashboards etc  | ✓ | 
| Task execution guarantees for business unit/application | ✕ |
| Shared authentication settings | ✓ |
| Synchronized projects, runbooks, dashboards etc | ✕ |
| Supports geographically disperse business units | ✕ |
| Robust RBAC support | ✓ |