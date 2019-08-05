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

## Why

In July 2018 we launched Octopus Cloud, our hosted version of Octopus, for customers who prefer to use an online version of Octopus without worrying about the infrastructure to run it. Octopus Cloud currently runs in Amazon Web Services, and each customer's instance executes in a dedicated virtual machine with a mix of other infrastructure. Rather than rebuild our entire product from the ground up as a multi-tentanted SaaS product, we decided the simplest MVP would be to run each customer on their own VM. This structure allowed us to bring the product to market quickly, and it provided an isolated, secure and stable solution for our customers. We [wrote about its architecture](https://octopus.com/blog/building-the-octopus-cloud-in-aws) if you're interested in learning more. The trade-off was that it was quite expensive to run, we had various stability problems which required our team to respond day or night, and we regularly hit the limits of AWS's services and had to request changes. We were proud to bring this service to market, but it was clear we needed to iterate. 

Our goal for Octopus Cloud 2.0 was to maintain the fantastic aspects of our the first iteration but also reduce it's running costs, reduce the support requirements and improve its scalability. 

## Journey

This journey started in January 2019, and we're nearing completion of it. Don't worry; we'll be writing all about it. As noted about, our first iteration of Hosted Octopus partly involved building our VM orchestration system to ensure instances are always running and being monitored. Seeing how far Docker has come in the past few years, we have decided to shift our architecture run Octopus instances in containers managed in Kubernetes clusters. Why reinvent the wheel if others have created a wheel that is balanced, auto-provisioning and self-healing?

![](ship.png)

### Windows Containers
Since Octopus runs as a .NET Full-Framework application, we originally explored running Octopus in Windows clusters. At the time Windows containers were _far_ too flakey for us to rely on for running services that our customers use. Although things have improved slightly since, during our testing back in mid-2018 the networking stack constantly fell to peices and there was little support for actually hosting Windows containers by any of the cloud vendors (save for self-managing the clusters using [AKS Engine](https://github.com/Azure/aks-engine)). The whole point of moving to Kubernetes was to offload most of the underlying infrastructure management and the payoffs for running the Windows containers did not look they would pay off. But there was another way... Linux containers.

### Linux & Netcore
For obvious reasons, running Linux containers in Kubernetes is a much more mature ecosystem and with Kubernetes coming out victorious of the orchestration wars of '17, most cloud vendors now have native support for Kubernetes as a Service. The only catch for this is that we would need to convert the Octopus Server process to .NET Core.

Our first _"wouldn't-it-be-crazy-if"_ proof of concept attempt of simply updating all project files to convert `<TargetFramework>net462</TargetFramework>` to `<TargetFramework>netcoreapp2.2</TargetFramework>`, showed that we had a bit of work to do and needed to be more methodical in our approach. Our second _"slash-and-burn"_ POC approach of commenting or pulling out huge chunks of code got us to a working version of Octopus Server than run (by some stretch of definition "run") on .NET Core natively inside a Linux container. With our code changes 80% of the way done we just had left the final 20% to complete which for those of you working in IT know should take only an additional one quarter of the time to acheive right? (Spoiler, it didnt).

Rather than completely split the code into two branches with a .NET Full Framework branch for on-premise and .NET Core for hosted, we wanted to simply cross-compile the Octopus Server solution for both frameworks. One of the results of producing software for customers to run as opposed to for our own internal systems, is that we need to do the utmost to keep supporting compatability on all environments that our customers might be supporting. Due to various library and code changes however this has required upgrading the Full Framework version from .NET 4.52 to .NET 4.72 that had the knock-on effect of dropping support for some older versions of windows  ([Windows Server 2008 EOL - Hello Linux](https://octopus.com/blog/windows-server-2008-eol-hello-linux)). 

Although we were now compiling for both frameworks from same code base, we still needed to make sure that we had the same level of trust in the code that we do for Full Framework builds. We have various levels of testing from unit to end-to-end across various Windows versions from Windows 2008 R1 to Windows 2019 and the last thing we wanted to do was add another "framework" dimension to this test matrix which would increase the test run time and complexity so we needed to be more methodical in what we were testing and where. Since Octopus Server at the moment exists largely for our hosted requirements, we decided to currently only add additional tests targeting the same Debian-based platform used in production. In addition since many of our E2E tests are more about flexing how the various components of the system interact, rather than small snippets of code that we examine in Unit testing, we decided that some tests were platform agnostic and would add no value executing across multiple platforms. 

The end outcome of all this is that we can now run an Octopus Server process on Linux for use in our hosted offering which will make this SaaS side of Octopus more commercially viable and simpler to maintain. It has forced our hand in working on other features that we expect to benefet even self hosted customers in future such as dynamically provisioned workers and a more stateless (and therefore stable) architecture.

### Where To Next
Although there is some additional work that will be required before we can provide the .NET Core version of Octopus Server for general usage, at some point we expect to make it publically available for customers to run on Linux, much like the anticipated release of [Linux Tentacles](https://octopus.com/blog/tentacle-on-linux). This project has been quite the learning experience and making Linux more part of our daily development process has resulted in providing a better product overall to our customers by having a broader understanding of different deployment environments. 