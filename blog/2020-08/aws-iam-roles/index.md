---
title: A look at using AWS IAM roles in Octopus
description: IAM roles allow users to temporarily assume new permissions or perform work from an EC2 instance without any additional credentials.
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

Managing credentials for cloud providers is a challenge, especially when you consider that you won't have the luxury of physical security, meaning one leaked admin key could grant access to your entire account from anywhere in the world. Nor are cloud accounts immune from the preverbal "rf -rf" scenario where an admin account accidentally deletes resources they shouldn't.

IAM roles can be used to provide task specific authorization, and when a role is assigned to an EC2 instance, users with access to that VM can inherit the role.

In this blog post we'll take a look at IAM roles in AWS and learn how they can be used in Octopus.

## Creating a role