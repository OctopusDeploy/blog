---
title: Introducing Octopus Server to Linux
description: Introducing Octopus Server to Linux, why we went down this path and its benefits. 
author: rob.erez@octopus.com
visibility: private
bannerImage: 
metaImage: 
published: 2019-08-01
tags:
 - Octopus Server
 - Linux
---

## Why

In July 2018, we launched Octopus Cloud, our hosted version of Octopus, for customers who prefer to use an online version of Octopus without worrying about the infrastructure to run it. Octopus Cloud runs in Amazon Web Services, and each customer's instance ran in a dedicated virtual machine and a mix of other infrastructure. We [wrote about its architecture](https://octopus.com/blog/building-the-octopus-cloud-in-aws) if you're interested in learning more. This structure allowed us to bring the product to market quickly, and it provided an isolated, secure and stable solution for our customers. The trade-off was that it was quite expensive to run, we hit xxx problems which required our team to respond day or night, and we regularly hit the limits of AWS's services and had to request changes. We were proud to bring this service to market, but it was clear we needed to iterate. 

## Goal

Our goal for Octopus Cloud 2.0 was to maintain the fantastic aspects of our the first iteration but also reduce it's running costs, reduce the support requirements and improve its scalability. 

## Journey

This journey started in January 2019, and we're nearing completion of it. Don't worry; we'll be writing all about it. Our architecture is shifting to running Octopus in Docker containers managed in Kubernetes clusters. We explore running Octopus in Windows clusters, but we encountered numerous problems which prompted us to explore retargeting Octopus Server to NETCORE so it could be run on commodity Linux infrastructure and meet our goals. This exploration was successful and Octopus 2019.7 shipped with the result of it. 

## Benefits

The benefits of running Octopus Server on Linux are quite compelling. 

1. Leverage **Linux Container support and scalability**. Docker support originated with Linux Container hosts, and they are stable and well supported. Microsoft has been doing great work to bring Windows into this ecosystem, but it didn't meet our needs. 
2. Unlock **cheap commodity infrastructure**. Numerous online Linux hosts offer competitive pricing, and this makes it very cheap to run a dedicated Octopus Server to handle all of your deployments. 
3. **Simpler setup and administration**. This point is a big one for us as we try to improve the stability of Octopus Cloud, reduce downtime for our customers and reduce on-call requirements by our team. We want our customers to be happy, and we want our team to sleep well instead of waking up to fix cloud infrastructure errors.

# Wrap-up

Octopus Server is coming to Linux, and the process has been fascinating. As a result, we're kicking off a series of posts where we discuss what it was like to retarget an application as large as Octopus (hint: the repo currently has xx commits and something around 100k lines of code). 
