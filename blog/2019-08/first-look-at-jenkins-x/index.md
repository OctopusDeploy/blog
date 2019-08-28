---
title: A first look at Jenkins X
description:
author: matthew.casperson@octopus.com
visibility: private
published: 2019-08-29
metaImage:
bannerImage:
tags:
 - Octopus
---

As a free and open source build server, Jenkins is [used by millions](https://www.cloudbees.com/press/jenkins-community-announces-record-growth-and-innovation-2017), so most developers have either used or at least heard about Jenkins. Like most build servers, Jenkins is typically installed on a server to consume source code, execute a build process on build agents, and either deploy or publish the resulting artifact.

Conceptually this model of a build server was easy for me to understand, and it applies to most of the popular solutions available today like Team City, Azure DevOps, Bamboo etc. So when it came time to working with [Jenkins X](https://jenkins-x.io/about/) for the first time, naturally I tried to understand it with the same conceptual model. This turned out to be a mistake.

I genuinely struggled to understand what Jenkins X was, but after some trial and error I went back and reread the [Jenkin X About Page](https://jenkins-x.io/about/what/). Two sentences buried deep in the text are critical to understanding Jenkins X:

> Jenkins X is opinionated.

> The critical thing to note is that you need to clear your mind from any Jenkins experience you might already have.

Internalizing these two statements is essential to appreciating what Jenkins X is. With that in mind, let's take a high level look at Jenkins X.

## It starts with the Kubernetes cluster

In a literal sense, Kubernetes is a container orchestrator. You describe the containers you want to run, how they communicate with each other, what resources they need, and Kubernetes does the hard work of running everything. Typically, Kubernetes hosts long running applications like web servers that continually wait for requests, and one of the great features of Kubernetes is that it will monitor these long running processes and restart them if they fail.

This traditional view of Kubernetes is the first thing you have to forget. To understand Jenkins X, think of Kubernetes more like a cloud operating system.

Just as you would use `apt-get` to install an application in Linux, Kubernetes can install applications using [Helm](https://helm.sh/).

Just as you would run short lived commands to build your applications and Docker images with the `docker` CLI, Kubernetes can build software and Docker images with [Skaffold](https://github.com/GoogleContainerTools/skaffold).

Just as you would host Docker image or other image artifact repositories like [Nexus](https://www.sonatype.com/nexus-repository-sonatype) with as services to be booted with the OS, so too can you deploy these same applications to Kubernetes.

::hint
Jenkins X configures your Kubernetes cluster with an opinioned set of open source tools that allow applications to be built and deployed to Kubernetes.
::

By installing Jenkins X, you will have a self contained Kubernetes cluster complete with a selection of hand picked and custom configured services ready to start building and deploying applications.

## Then extends to your local development environment

I'm used to my local development environment being completely separate from the central build server. To get my code into the build server I would typically push it to a central GIT repository, go to the CI server, point a project at the GIT repo, configure the build and then kick it off.

If my CI server offered a CLI tool, it was exclusively for managing the CI server. Such tools don't have any concept of the code I am working on.

Again, forget everything you have learned with traditional CI servers. Jenkins X has opinions about how your code is built and deployed, and is not shy about configuring everything for you.

To have your exiting code built by Jenkins X you will run `jx import`. Running this command will add several files to your project including `jenkins-x.yml` (which is the [Jenkins X pipeline](https://jenkins-x.io/architecture/jenkins-x-pipelines/)), `Dockerfile` (which builds the [Docker image](https://docs.docker.com/engine/reference/builder/)), `skaffold.yaml` (which is the [Skaffold project configuration](https://skaffold.dev/docs/references/yaml/)) and a `charts` directory (which provide the [Helm chart template](https://helm.sh/docs/chart_template_guide/)).

Jenkins X will then place all the code into a GIT repository, push it, and start the build.

As you can see from the output below, Jenkins X is tying together a number of different tools and services to create a repeatable build process.

```
PS C:\Users\Matthew\Downloads\ThymeleafSpring> jx import
WARNING: No username defined for the current Git server!
? Do you wish to use mcasperson as the Git user name: Yes
The directory C:\Users\Matthew\Downloads\ThymeleafSpring is not yet using git
? Would you like to initialise git now? Yes
? Commit message:  Initial import

Git repository created
selected pack: C:\Users\Matthew\.jx\draft\packs\github.com\jenkins-x-buildpacks\jenkins-x-kubernetes\packs\maven
? Which organisation do you want to use? mcasperson
replacing placeholders in directory C:\Users\Matthew\Downloads\ThymeleafSpring
app name: thymeleafspring, git server: github.com, org: mcasperson, Docker registry org: kubernetes-demo-198002
skipping directory "C:\\Users\\Matthew\\Downloads\\ThymeleafSpring\\.git"
Using Git provider GitHub at https://github.com
? Using organisation: mcasperson
? Enter the new repository name:  ThymeleafSpring
Creating repository mcasperson/ThymeleafSpring
Pushed Git repository to https://github.com/mcasperson/ThymeleafSpring
Creating GitHub webhook for mcasperson/ThymeleafSpring for url http://hook.jx.35.194.232.107.nip.io/hook

Watch pipeline activity via:    jx get activity -f ThymeleafSpring -w
Browse the pipeline log via:    jx get build logs mcasperson/ThymeleafSpring/master
You can list the pipelines via: jx get pipelines
When the pipeline is complete:  jx get applications

For more help on available commands see: https://jenkins-x.io/developing/browsing/
```

::hint
Jenkins X configures your code and repository with an opinioned set of build tools and hooks to establish a continuous build pipeline. 
::
