---
title: Deprecating Mono
description: Octopus Deploy is planning to drop support for deploying to SSH targets via Mono. This post talks through the background for this decision and what users need to do to move to utilize .net core.
author: robert.erez@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - Linux
----

This post will walk through the history of Mono at Octopus, along with the steps that we have taken towards removing it as a dependancy for Linux deployments.
If your Octopus Server instance is configured to deploy to SSH targets via Mono (and not self-contained .Net Core)

A [GitHub issue has been created](https://github.com/OctopusDeploy/Issues/issues/8146) where you can provide feedback or questions to reduce the potential impact.

## Background - When and why does Octopus use Mono?
Way back in 2015, the release of Octopus 3.0 introduced support for [deploying releases on Linux via SSH Targets](https://octopus.com/blog/deployment-targets-in-octopus-3#multiple-types-of-machines-deployment-targets). The Octopus Deploy deployment execution engine framework, Calamari, was at the time built only on .net full framework. .NetCore had only recently been announced and 1.0 was yet to be released over a year later so the only way to run the deployment process was invoking it via [Mono](https://www.mono-project.com/docs/about-mono/). Mono has served it's purpose well, however with .netcore f..........

### Technical Notes - What is Mono?
Mono provides a [CLI](https://en.wikipedia.org/wiki/Common_Language_Infrastructure) (Common Language Infrastructure) virtual machine which can be run on various non windows platforms. The CLI comprises of the runtime required to invoke .Net code  previously into CIL (Common Intermediate Language)
including components such as the JIT compiler, garbage collector and library loader. These components would typically be provided by the installed .Net Framework installed on a Windows machine. It is an amazing set of tools that was developed by the whole Mono team that opened up .Net development for (and on) non Windows machines before it was fully supported by Microsoft itself.
 
The Mono CLI also includes a .NET Framework Class Library which, although compatible with Microsoft’s .NET Framework classes, are seperate to the standard .Net Framework libraries used on a typical Windows machine using .Net Full Framwork. This mean that these libraries are tied to specific versions of the Full Framework and are not intended to serve as a replacement for libraries found in .Net Core. One side effect of this is that code being compiled for Full Framework, Mono and .Net Core are filled with various compiler directives switching out various import statments or even entire classes. 

### The rise of .Net Core and reducing dependency on Mono
For some users the Mono dependency was problematic, so in [early 2017 we introduced support for what we call Raw Octopus](https://octopus.com/blog/trying-raw-octopus). This mechanism allows for running scripts directly through the SSH shell without any of the helpful orchestration provided by Calamari, but also as a result without any of the dependencies required to runn Calamari. Due to it's simplicity this mechanism has always remained a niche fallback mechanism available for constrained platforms.

The supported API surface area provided by the release of .netcore 2.0 in mid 2017 allowed us to quickly work towards [Mono-less SSH targets in 3.16](https://octopus.com/blog/octopus-release-3-16#ssh-targets-sans-mono). This was the first releas that allowed for running Octopus deployments on a Linux system with .netcore and was fairly quickly taken up by customers as a simpler mechanism for running their deployment workloads on Linux servers.




## Why are we deprecating mono?
### Development Costs
### Support Impacts
### Risk Reduction and Support Improvements


## Moving to .netcore SSH Targets
### Target Dependencies
### Updating your Octopus Target

### Current limitations
* ScriptCS
* FSharp
* Lack of support for RHEL6, and Fedora

## Conclusion

Close off the post by restating the main points of the post, share any closing thoughts, and invite feedback.

## Learn more

- [link](https://www.example.com/resource)
