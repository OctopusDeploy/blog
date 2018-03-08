---
title: "Integration 101: Octopus and Build Servers"
description: "A brief introduction on how to approach your brand new integration between Octopus and your Build Server"
author: dalmiro.granas@octopus.com
visibility: private
published: 2018-03-14
metaImage: metaimage-cloudsaving.png
bannerImage: blogimage-buildserver.png
tags:
 - Integration
 - TeamCity
 - Team Foundation Server

---

In my 3 years providing support to our users in Octopus, *Integrating Octopus with build servers* is probably the subject I answered the most questions about. With this blog post I'm going to try to give a few tips on how you should approach this task if you are starting it from scratch, regardless of the build server technology you are using. 

This blog post is aimed for users that are just starting their journey integrating Octopus into their CI pipeline. If you already have this up and running, there might not be too much value here for you :)

## In this post

!toc

## Separation of Concerns

In a way, Octopus and every build server technology out there can seem pretty similar at a first glance. A few things in common they have are:

- Both tools belong in CI pipelines and are aimed to help development teams getting their code out there faster and more reliably.
- Both are basically process runners. You define a Build/Deployment process filled with steps, you execute it and magic happens.

But that's pretty much where the similarities stop. If you want to integrate any 2 tools, you need to have a clear image of what each tool is going to be

### What should the build server do?

- **Compile your binaries**. This means running `MSBuild`,`NPM`,`javac`,`dotnet.exe`, etc, and dropping your compiled app to a folder on the build agent.
- **Run tests**. While its true that you can run these anywhere, because at the end of the day its just a CLI app executing tests from a fixture file, most build servers nowadays come with features that improve the overall testing experience quite a lot. Mostly UI sweetness like showing you how many tests failed right next to your build status, or telling you if the test that failed is a new one or not.
- **Pack and Push your app to a repository**.  

### What should Octopus do?



