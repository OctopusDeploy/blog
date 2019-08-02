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

In July 2018, we launched Octopus Cloud, our hosted version of Octopus, for customers who prefer to use an online of Octopus without managing infrastructure. Octopus Cloud runs in AMazon Web Services and each customer gets their own virual machine. This is great for isolation, scalabilty and something else but it's also quite expensive to run. So we embarked on a second iteration of Octopus Cloud to provide a stable, scalable solution for our customers but in a more cost effective way. Our goal was to blah blah blah. 

## Journey

This journey started in January 2019 and we're nearing completion of it. Don't worry, we'll be writing all about it. Our architecture has shifted to running Octopus in docker containers managed in Kubernetes clusters. We explore running Octopus in windows clsuters but encountered numerous problems which steered us to explore retargetting Octopus Server on NETCORE so it could be run on commondity linux infrastructure and meet our goals. This exploration was successful and Octopus 2019.7 shipped with the result of it. 

## Benefits

The benefit of this are 1, 2, 3, 


This has been a very interesting process to say the least and as a result, we're kicking of a series of posts where we discuss what it was like to retarget an application as large as Octopus (hint: the repo currently has xx commits and something around 100k lines of code). 

# Wrap-up

So we introduced Octopus Server on Linux as 