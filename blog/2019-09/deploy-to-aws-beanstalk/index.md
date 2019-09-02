---
title: Deploying to AWS Elastic Beanstalk with Octopus
description: This blog post looks at the process of deploying a .NET Core application to AWS Elastic Beanstalk.
author: matthew.casperson@octopus.com
visibility: private
published: 2020-01-01
metaImage:
bannerImage:
tags:
 - Octopus
---

Elastic Beanstalk is a Platform as a Service (PaaS) offering provided by AWS that allows developers to deploy code written in a variety of languages such as .NET, Java, PHP, Node.js, Go, Python and Ruby onto preconfigured infrastructure. You simply upload your application, and Elastic Beanstalk automatically handles the details of capacity provisioning, load balancing, scaling, and application health monitoring.

The lifecycle of a Beanstalk application shares much in common with Octopus. Beanstalk defines the idea of an application being deployed to multiple isolated environments, with each environment potentially having unique settings. The overlap means Octopus can add a lot of value to deployments in Beanstalk.

However Beanstalk has some unique requirements that we need to take into account in order to properly integrate it with Octopus. In this blog post we'll explore a sample deployment script that allows Octopus to deploy a .NET Core application to AWS Elastic Beanstalk.
