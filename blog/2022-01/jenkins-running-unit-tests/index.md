---
title: Running unit tests in Jenkins
description: Learn how to run unit tests in Jenkins and capture the results
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

Verifying code changes with unit tests is a critical process in the development workflow. Jenkins provides a number of plugins to collect and process the results of tests allowing developers to browse the results, debug failed tests, ignore flakey or faulty tests, and generate reports on the history of tests over time.

In this post you'll learn how to add unit tests to a Jenkins project and configure plugins to process the results.

## Prerequisites

To follow along with this post you'll need Jenkins instance. The [Traditional Jenkins Installation](/blog/2022-01/jenkins-install-guide/index.md), [Docker Jenkins Installation](/blog/2022-01/jenkins-docker-install-guide/index.md), or [Helm Jenkins Installation](/blog/2022-01/jenkins-helm-install-guide/index.md) guides provide instructions on installing Jenkins in your chosen environment.

The sample application's you'll build are written in Java and DotNET Core, so the Java Development Kit (JDK) and DotNET Core SDK must be installed on the Jenkins controller or agents that perform the builds.

You can find instructions on installing the DotNET Core SDK from the [Microsoft website](https://dotnet.microsoft.com/download/dotnet/3.1). The sample project is written against DotNET Core 3.1.

The OpenJDK project provides a free and open source distributions that you can use to compile Java applications. There are many OpenJDK distributions to choose from including [OpenJDK](https://openjdk.java.net), [AdoptOpenJDK](https://adoptopenjdk.net), [Azul Zulu](https://www.azul.com/downloads/), [Red Hat OpenJDK](https://developers.redhat.com/products/openjdk/download), and more. I typically use the Azul Zulu distribution, although any distribution will do.