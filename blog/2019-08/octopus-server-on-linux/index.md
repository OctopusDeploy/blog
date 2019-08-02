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

In July 2018, we launched Octopus Cloud, our hosted version of Octopus, for customers who prefer to use an online version of Octopus without worrying about the infrastructure to run it.  Octopus Cloud runs in Amazon Web Services, and each customer's instance ran in a dedicated virtual machine and a mix of other infrastructure. We [wrote about its architecture](link to post on octopus cloud 1.0 arch) if you're interested in learning more. This structure allowed us to bring the product to market quickly and it provided an isolated, secure and stable solution for our customers. The trade-off was that it was quite expensive to run, we hit xxx problems which required our team to respond day or night, and we regularly hit the limits of AWS's services and had to request changes. 

## Goal

So once Octopus Cloud stabilised, and we were confident with its operation, we embarked on a second iteration. Our goal for Octopus Cloud 2.0 was to maintain the positive aspects of our the first iteration but also reduce it's running costs, reduce the support requirements and improve its scalability. 

## Journey

This journey started in January 2019 and we're nearing completion of it. Don't worry, we'll be writing all about it. Our architecture has shifted to running Octopus in docker containers managed in Kubernetes clusters. We explore running Octopus in windows clsuters but encountered numerous problems which steered us to explore retargetting Octopus Server on NETCORE so it could be run on commondity linux infrastructure and meet our goals. This exploration was successful and Octopus 2019.7 shipped with the result of it. 

## Benefits

The benefit of this are 1, 2, 3, 


This has been a very interesting process to say the least and as a result, we're kicking of a series of posts where we discuss what it was like to retarget an application as large as Octopus (hint: the repo currently has xx commits and something around 100k lines of code). 

# Wrap-up

So we introduced Octopus Server on Linux as 