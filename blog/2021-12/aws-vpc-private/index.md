---
title: Create a Private AWS VPC with CloudFormation
description: Learn how to create a private AWS VPC with this sample CloudFormation template
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

Virtual Private Clouds, or VPCs, are the backbone of any infrastructure deployed to AWS. Almost all resources require a VPC, and most resource segregation is done using VPCs.

Unfortunately, despite their ubiquity, creating VPCs is not quite as straight forward as they could be. In this post you'll learn the different types of VPCs available in AWS, and find an example CloudFormation template that used to deploy a simple VPC with private subnets.

## Types of subnets

AWS has two types of subnets: public and private.

A private subnet has no connection to the internet. Resources in a private subnet must have private IP addresses, and can only communicate with resources in other subnets within the same VPC.

A public subnet has a connection to the internet via an [internet gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html). An internet gateway is defined by AWS as:

> a horizontally scaled, redundant, and highly available VPC component that allows communication between your VPC and the internet. 



