---
title: Manage all the consumers of your JavaScript library using Octopus Tenants
description: Use Octopus tenants to control who references which version of your JavaScript library project.
author: lee.meyer@octopus.com
visibility: public
published: 
metaImage: 
bannerImage: 
bannerImageAlt: 
isFeatured: false
tags:
 - DevOps
 - AWS
 - Tenants
---

In my [previous post](https://octopus.com/blog/deploying-javascript-library-project-with-octopus), you learned how to use Octopus to deploy a hash-named JavaScript library bundle to cloud storage, where it could be referenced by other projects via an automatically updated variable within a [Library Variable Set](https://octopus.com/docs/projects/variables/library-variable-sets). That's a solid start for managing reusable front-end code in Octopus. It keeps every consumer of your bundle on the latest version of your library as releases happen, which might be the exact behavior you need, especially if you have only a few internal projects referencing a small or medium-sized JavaScript library. However, if you continue following this pattern as your organisation scales up, you might run into certain dilemmas.

### What if a team using your library needs to deploy an update to their project, but doesn't want to retest against the latest version?

This might happen when a hotfix is needed. As your front-end library grows, you won't want to rush upgrading it for the sake of releasing an unrelated fix. You could override the Library Variable at the project level. This would be a reasonable solution once in a while, but if it's part of normal development flow, and multiple teams reference your script and want to be able to upgrade when it's convenient, then easy visibility into which consumer is on which version of your library is helpful. You'd also like to be able to roll back a change that inadvertently broke one consumer, or upgrade some website that's failing due to a known bug in an old version of your library. In these situations, you want to release not only to a specific environment, but also to a specific set of consumers.  

### What if the consumer of your library isn't an internal project?

Maybe you created a widget that users add to their website by copy-pasting a code snippet from your website. Or a library started life as internal, but turned out to be general-purpose enough for your company to make it available to the world via a CDN. In these cases, it becomes impossible for you to update the HTML references to your script, so you have to separate the concept of releasing your JavaScript and releasing the code that uses it, and you will have to solve cache busting in a new way.

## Tenants to the rescue

[Tenants](https://octopus.com/docs/tenants) are a good conceptual fit for representing consumers of your JavaScript library. Tenants are often used to represent customers of your application. Your customers in this case might be internal, but here at Octopus we have found that scaling up our engineering team has shown it's helpful to think of other teams as internal customers. You are updating script dependencies on different websites instead of deploying software to different customers' infrastructure, but all the requirements for how you'd like to manage that in Octopus are the same as if you were deploying appropriate versions of a server-side app for different customers. Here's an example of the dashboard end up with after following the instructions in this post, if you were deploying your shared script to a company WordPress blog, an external customer, and your company's main website.

![bundle tenants overview](bundle-tenants.gif)



