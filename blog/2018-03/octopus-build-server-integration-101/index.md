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

In my 3 years providing support to our users at Octopus, *Integrating Octopus with build servers* is probably the subject I answered the most questions about. With this blog post I'm going to try to give a few tips on how you should approach this task if you are starting it from scratch, regardless of the build server technology you are using. 

This blog post is aimed for users that are just starting their journey integrating Octopus into their continuous integration (CI) pipeline. If you already have this up and running, there might not be too much value here for you :)

## In this post

!toc

## Separation of Concerns

In a way, Octopus and every build server technology out there can seem pretty similar at a first glance. A few things in common they have are:

- Both tools belong in CI pipelines and are aimed to help development teams getting their code out there faster and more reliably.
- Both are basically process runners. You define a Build/Deployment process filled with steps, you execute it and magic happens.

But that's pretty much where the similarities stop. If you want to integrate any 2 tools, you need to have a clear image of which task is each tool going to take care of. 

The below are two (very simplified) lists of what each tool's role should be in an ideal CI pipeline.

### What should the build server do?

- **Compile your binaries**. This means running `MSBuild`,`NPM`,`javac`,`dotnet.exe`, etc, and dropping your compiled app to a folder on the build agent.
- **Run tests**. While its true that you can run these anywhere, because at the end of the day its just a CLI app executing tests from a fixture file, most build servers nowadays come with features that improve the overall testing experience quite a lot. Mostly UI sweetness like showing you how many tests failed right next to your build status, or telling you if the test that failed is a new one or not.
- **Pack and Push your app to a repository**. Once the tests passed, create a package with the output of your build and push it to your repository.
- **Call Octopus and tell it to create a Release/Deployment**

###What should Octopus do?

- **Provision infrastructure (optional)**. If you need to create an `Azure WebApp`, an `Elastic Beanstalk` instance, scale up a VM set, create an IIS/Tomcat website or anything that's related to setting up **the place** where you'll be putting your compiled code, adding it as one of the first steps in your Octopus deployment process is a very good idea. //maybe mention a thing our two about our awesome infrastructure as code story.

- **Set configuration values in your application before deployment**. Ideally, the content of your package should be deployable to *any* environment in your pipeline, and the only thing that should be different are environment-specific configuration values such as connection strings, passwords, keys to connect to other services, etc. Octopus has a wide set of feature to deal with any configuration file modification that's needs to be done *before* pushing your application to its destination.

- **Deploy your Application**.

  ​

##So how should I start integrating Octopus into my CI pipeline?

One of the approaches that I've seen has helped many of our customers, was to split this task into 3 stages, while keeping in mind the separation of concerns mentioned previously. We are going call these 3 stages **The Build**, **The Deployment** & **The Glue**. Each stage will have its own **end goal** that we are going to focus on.

Now, before we dwelve into each of the stages, I want to mention that stages 1 and 2 can we worked on in any order, because they won't be touching each other until we reach stage 3. This means that if you are working with a teammate on this integration, he could be focusing in stage 1 while you focus in stage 2 (and you could even bet a beer to see who finishes first).

### Stage 1: The build

**End goal:** By the end of this stage you should end up with a package of [any of the supported formats](link) that contains your build output. The package will only be sitting in a folder on your build agent for now.





