---
title: Octopus Server 2019.9 with Long Term Support (LTS)
description: Octopus Server 2019.9 LTS is the fourth release with six months of long-term support. We recommend this release for self-hosted customers.
author: rob.pearson@octopus.com
visibility: private
bannerImage: blogimage-ltsrelease.png
metaImage: blogimage-ltsrelease.png
published: 2019-10-09
tags:
- Product
---

![Cars on slow lane and fast lane](blogimage-ltsrelease.png)

<h2>Octopus Deploy 2019.9 LTS</h2>

Octopus Deploy `2019.6 LTS` is now available for teams running Octopus Deploy self-hosted, and we recommend this release for our self-hosted customers. Our [long-term support (LTS) program](https://octopus.com/docs/administration/upgrading/long-term-support) includes releases with six months of support, including critical bug fixes and security patches. They do not include new features, minor enhancements, or minor bug fixes; these are rolled up into the next LTS release.

<a href="https://octopus.com/downloads" class="btn btn-primary btn-lg">Download now</a>

This is our third release with six months of long term support, and the following table shows our current LTS releases.

| Release               | Long term support           |
| --------------------- | --------------------------- |
| Octopus 2019.9        | Yes                         |
| Octopus 2019.6        | Yes                         |
| Octopus 2019.3        | Expired                     |

Keep reading to learn about what's in this release and any breaking changes.

<h2>In this post </h2>

!toc

## Streamlined deployment process editor



## Tentacle for Linux

![Linux Tentacle configuration](linux-tentacle.png "width=500")

This release also includes early-access for our upcoming Linux Tentacle. [Octopus 3.0](https://octopus.com/blog/deployment-targets-in-octopus-3) introduced support for Linux deployments over SSH; however, in highly secure environments inbound ports cannot be opened on production servers. Our Linux Tentacle agent solves this security concern with support for communication between the Octopus Server and Linux deployment targets in listening and polling modes. Polling mode specifically removes the requirement for open ports as the polling Tentacle establishes communication with the Octopus Server.

[Learn more](https://octopus.com/docs/infrastructure/deployment-targets/linux/tentacle)

## Cloning Tenants

## Other improvements



## Breaking changes

This release includes a single breaking change as we dropped support for Windows Server 2008 SP2. We covered this change 

* Octopus 

## Wrapping up

Octopus Server 2019.9 is now available and we recommend . Happy long-term deployments!
