---
title: Running end to end tests in Jenkins
description: Learn how to run end to end tests in Jenkins and capture the results
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

End-to-end (E2E) tests represent the of the final stages of automated testing. E2E are long running (certainly with respect to unit tests that can complete thousands of tests in seconds), and are typically executed by external tools which interact with the application under test through public interfaces like web pages or HTTP APIs.

In this post you'll learn how to run E2E tests with Cypress, to validate interactions with web pages, and Postman, to validate HTTP APIs.

## Prerequisites

To follow along with this post you'll need a Jenkins instance. The [Traditional Jenkins Installation](/blog/2022-01/jenkins-install-guide/index.md), [Docker Jenkins Installation](/blog/2022-01/jenkins-docker-install-guide/index.md), or [Helm Jenkins Installation](/blog/2022-01/jenkins-helm-install-guide/index.md) guides provide instructions to install Jenkins in your chosen environment.

The sample applications you'll build are written in Java so the Java Development Kit (JDK) must be installed on the Jenkins controller or agents that perform the builds.

The OpenJDK project (and its downstream projects) provide free and open source distributions that you can use to compile Java applications. There are many OpenJDK distributions to choose from including [OpenJDK](https://openjdk.java.net), [AdoptOpenJDK](https://adoptopenjdk.net), [Azul Zulu](https://www.azul.com/downloads/), [Red Hat OpenJDK](https://developers.redhat.com/products/openjdk/download), and more. I typically use the Azul Zulu distribution, although any distribution will do.

Cypress must be installed on the Jenkins controller or agents to run browser based E2E tests. The [Cypress documentation](https://docs.cypress.io/guides/getting-started/installing-cypress) provides instructions for installing Cypress.

Newman is the command line tool for running Postman tests and must be installed on the Jenkins controller or agents to run API E2E tests. The [Postman documentation](https://support.postman.com/hc/en-us/articles/115003703325-How-to-install-Newman-) has instructions for installing Newman.