---
title: Why we bet on Kubernetes 
description: Why we chose Kubernetes to host Octopus Cloud 
author: michael.richardson@octopus.com
visibility: private
published: 2020-01-01
metaImage:
bannerImage:
tags:
 - Octopus
---

Last year we released Octopus Cloud, our hosted software-as-a-service version of Octopus Deploy.

Octopus had always been designed to be hosted on user's own hardware, not as a multi-tenant co-hosted solution. So when architecting Octopus Cloud there were many different paths available.  In accordance with the finest of engineering traditions, we started with the Simplest Thing That Could Possibly Work, which in this case was hosting each customer on a dedicated virtual machine.

This decision traded an unpredicatable cost (implementing a more advanced solution) for a predictable cost (AWS bills).

- We didn't have much use for Octopus
- Only Octofront
- Then Octopus Cloud
- v1
- v2 options 
- The decision: .NET Core, linux, containers and kubernetes