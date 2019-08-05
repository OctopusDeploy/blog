---
title: Introducing Octopus Server to Linux
description: Introducing Octopus Server to Linux, why we went down this path and its benefits. 
author: robert.erez@octopus.com
visibility: private
bannerImage: 
metaImage: 
published: 2019-08-01
tags:
 - Octopus Server
 - Linux
---

With Octopus Deploy 2019.7 we shipped changes that retarget Octopus Server to Microsoft's .NET Core framework and makes running Octopus Server on Linux possible. This post talks about why we embarked on this journey and the benefits of making this change.

## Octopus Cloud

In July 2018, we launched Octopus Cloud, our hosted version of Octopus, for customers who prefer to use an online version of Octopus without having to worry about the infrastructure to run it. Version 1.0 of Octopus Cloud runs in Amazon Web Services, and each customer has an instance that executes in a dedicated virtual machine. Rather than rebuild our entire product as a multi-tenant SaaS product, we decided the simplest minimum viable product (MVP) was to run each customer on their own VM. This structure allowed us to bring the product to market quickly, and it provided an isolated, secure, and stable solution for our customers. We wrote about [Octopus Cloud's architecture](https://octopus.com/blog/building-the-octopus-cloud-in-aws) if you're interested in learning more. This approach was quite expensive to run, it had numerous components that we had to support, and we regularly hit AWS service's limits. We are proud of bringing this service to market, but it was clear we needed to iterate.

## Octopus Cloud 2.0

Our goal for Octopus Cloud 2.0 is to maintain the fantastic aspects of our first iteration, reduce the running costs, make it easier to support, and improve the scalability. 

This work started in January 2019, and we're nearing the end of it. The first iteration of Octopus Cloud involved building our own VM orchestration system to ensure instances are always available and monitored, but seeing how far Docker has advanced in the last few years, we decided to shift our architecture to run Octopus Cloud instances in containers managed in Kubernetes clusters. After all, why reinvent the wheel if somebody has already created a balanced wheel that is auto-provisioning and self-healing?

### Windows Containers

Since Octopus runs as a .NET Full-Framework application, we originally explored running Octopus in Windows clusters. At the time, Windows containers were too flakey for us to rely on them to run services that our customers use. Although things have improved slightly since then, during our testing back in mid-2018 the networking stack wasn't reliable enough, and there was little support for hosting Windows containers with any of the cloud vendors. We could have self-managing the clusters using [AKS Engine](https://github.com/Azure/aks-engine), but one of the reasons for moving to Kubernetes was to offload most of the underlying infrastructure management, and the payoffs for running Windows containers did not look they would pay off. But there was another way... Linux containers.

### Linux & .Net Core

There is a mature ecosystem for running Linux containers in Kubernetes, with most cloud vendors offering native support for Kubernetes as a Service. The only catch for this is that we needed to convert the Octopus Server process to .NET Core.

For our first proof of concept, we updated all project files to convert `<TargetFramework>net462</TargetFramework>` to `<TargetFramework>netcoreapp2.2</TargetFramework>`. This showed us that we had a bit of work to do and needed to be more methodical in our approach. For our second POC approach, we commented out or pulled out huge chunks of code which got us to a _working_ version of Octopus Server that run on .NET Core natively inside a Linux container. 

This got us 80% of the way there, but the last 20% took longer than we expected.

Rather than completely split the code into two branches with a .NET Full Framework branch for on-premises and .NET Core for hosted, we wanted to cross-compile the Octopus Server solution for both frameworks. One of the results of producing software for customers to run as opposed to for our internal systems, is that we need to keep supporting compatibility on all environments that our customers might be supporting. Due to various library and code changes, however, this meant upgrading the Full Framework version from .NET 4.52 to .NET 4.72 that had the knock-on effect of dropping support for some older versions of Windows, for instance, see ([Windows Server 2008 EOL - Hello Linux](https://octopus.com/blog/windows-server-2008-eol-hello-linux)). 

Although we were now compiling for both frameworks from same code base, we still needed to make sure that we had the same level of trust in the code that we do for Full Framework builds. We have various levels of testing from unit to end-to-end across various Windows versions from Windows 2008 R1 to Windows 2019. The last thing we wanted was to add another "framework" dimension to this test matrix which would increase the test run time and complexity. We needed to be more methodical in what we were testing and where. Since Octopus Server at the moment exists largely for our hosted requirements, we decided to only add additional tests targeting the same Debian-based platform used in production. Since many of our E2E tests are more about flexing how the various components of the system interact, rather than small snippets of code that we examine in Unit testing, we decided that some tests were platform agnostic and executing across multiple platforms added no value. Instead of testing for the sake of testing, we made some pragmatic choices about what our code was doing, and what factors might make it act differently and therefore require testing.

The outcome of all this is that we can run an Octopus Server process on Linux for use in our hosted offering which makes the SaaS side of Octopus more commercially viable and simpler to maintain. It has forced our hand in working on other features that we expect will also benefit self-hosted customers in the future, such as dynamically provisioned workers and a more stateless (and therefore stable) architecture.

### Where To Next
Although there is some additional work that's required before we can provide the .NET Core version of Octopus Server for general usage, at some point we expect to make it publicly available for customers to run on Linux, just like the anticipated release of [Linux Tentacles](https://octopus.com/blog/tentacle-on-linux). This project has been a learning experience and making Linux more a part of our daily development process has resulted in providing a better product overall to our customers by having a broader understanding of different deployment environments. 

Happy deployments!