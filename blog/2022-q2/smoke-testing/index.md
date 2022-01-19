---
title: Smoke testing your infrastructure with runbooks
description: Learn how to use runbooks to perform high level smoke tests against your applications and infrastructure
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

*The internet is broken!*

Anyone who has spent some time on a help desk will have heard this, and other equally vague, descriptions of issues customers have run into. Getting actionable information is half the battle when diagnosing an issue.

When supporting complex infrastructure though, it can be hard to know how the system was designed, which is turn makes it hard to know what questions to ask and where to find information that may aid in resolving the issue. It is the nature of the kind of custom applications typically found in enterprise environments that each is a evolution of the last, or written by an entirely different team using a unique approach each time. This means the business knowledge around how any given application should be supported is often only found in the heads of a few employees.

Runbooks provide a way to capture this business knowledge in an automated and testable way, ensuring the support team can quickly diagnose high level issues and efficiently respond to customer requests. In this post you'll find an example runbook designed to smoke test a typical microservice application in AWS.

