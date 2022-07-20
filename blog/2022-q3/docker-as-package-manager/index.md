---
title: Using Docker as a package manager
description: This post demonstrates how Docker is a convenient solution for downloading and running many CLI tools
author: matthew.casperson@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: placeholderimg.png
bannerImage: placeholderimg.png
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - DevOps
---

A constant challenge when automating DevOps tasks is ensuring you have the correct tools for the job. Linux users have long enjoyed the ability to install tools from extensive and well maintained package repositories, while macOS users have HomeBrew and MacPorts, and Windows users have Chocolatey and winget. However, an increasing number of cloud based tools, such as platform specific CLI tools (eksctl, aws-iam-authenticator, kubectl, kind, helm, etc.) can only be reliably installed across multiple operating systems and Linux distributions via a direct binary download. Finding and downloading tools is an point of friction when automating DevOps tasks.

Docker provides a solution by allowing CLI based tools to be run from a Docker image. In this post, we'll explore how Docker can be used as a universal package manager to download and run many CLI tools across multiple operating systems.

## Find the right tools to provide DevOps solutions

DevOps is all about repeatability and automation, and frequently requires scripting tasks using specialized CLI tools. Many of the environments where scripts are run are ephemeral in nature. Examples include CI/CD agents that are created and destroyed as needed, Kubernetes pods used to run jobs, and virtual machines that scale up and down in response to demand. Often this means script writers can not assume CLI tools are installed, or depend on the particular version of a tool being available.

As we noted in the introduction, many of the tools DevOps teams require only provide direct downloads of the binaries. This means even a simple script may need to first find the appropriate download link for any required CLI tools, download them (usually with some kind of retry logic to deal with network instability), extract or install the tools, and then finally run them.

This task is complicated by the fact that the tools required to download files from the internet differ between operating systems. Linux and macOS users may be able to count on tools like `curl` or `wget` being installed, while Windows users will likely use PowerShell CmdLets.

But what if there was a better way?

## Docker as a universal package manager

Every major CLI tool and platform provides a well maintained Docker image these days. Whether the images are published by the vendors themselves on repositories like [Docker Hub](https://hub.docker.com/) or maintained by a third party like [Bitnami](https://bitnami.com/), there is a good chance you will find an up to date version of a CLI tool you need as a Docker image.

The nice thing about running Docker images is the commands to download an image and execute it are the same across all operating systems and for all tools.

To download an image, reusing any previously downloaded images and with automatic reties, run the command:

```bash
docker pull image-name
```

The image is then executed with the command:

```bash
docker run image-name
```

As we'll see in the next sections, there are additional arguments required to reliably run images for short lived managements tasks, but generally speaking these two commands are all you need to know for every tool and operating system. This is huge improvement over unique scripts between Windows and Linus to download binary files from custom URLs.

## A practical example

Let's look at helm as a practical example of running CLI tools from Docker images.

## Conclusion

Close off the post by restating the main points of the post, share any closing thoughts, and invite feedback.

## Learn more

- [link](https://www.example.com/resource)

Happy deployments! 
