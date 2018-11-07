---
title: Better Know a Step Template - Create and Clone Tenant
description: description
author: ryan.rousseau@octopus.com
visibility: private
bannerImage: blogimage-todo.png
metaImage: blogimage-todo.png
published: 2018-11-13
tags:
 - Step Template
 - Tenants
---

![Banner Image](blogimage-todo.png)

Hello there! It's Ryan from Octopus and today I'm starting a new blog series that I'm pretty excited about. In _Better Know a Step Template_ (yes, blatantly ~~stolen from~~ inspired by The Colbert Report), we'll pick a step template from our [Community Library](https://library.octopus.com) to highlight and discuss.

We have 316 (at time of writing) templates to better know and we are starting off with a bang, a double feature! We are getting to know two newly minted templates - [Create Tenant](https://library.octopus.com/step-templates/581e7211-c9e2-4d7b-8934-bcdac421d022/actiontemplate-create-tenant) and [Clone Tenant](https://library.octopus.com/step-templates/3b0f8df0-93b8-44eb-86dd-264d1283ae70/actiontemplate-clone-tenant)!

## Overview

Both of these templates were created to solve a specific problem. [Multi-tenancy](https://octopus.com/docs/deployment-patterns/multi-tenant-deployments) is a powerful feature of Octopus Deploy. The ability to deploy the same application to dozens or hundreds of customers is great! However, if you are deploying a suite of applications or a fleet of microservices, connecting a new tenant to all of those projects and environments can be tedious.

Now you have two quick and easy options provided through the Community Library. Let's look at Create Tenant first.

## Create Tenant

### Functionality

Both of these are pretty straightforward in the functionality department. Create Tenant does exactly what the name says, it creates a tenant on your Octopus server.

It shines with its ability to create the tenant with a set of [tenant tags](https://octopus.com/docs/deployment-patterns/multi-tenant-deployments/tenant-tags) and a full set of project and environment connects all in one step. As a step template, it can be executed directly from the library page or it can be set up in a deployment process with a [prompted variable](https://octopus.com/docs/deployment-process/variables/prompted-variables) connected to the tenant name. I prefer the second method because it means that I don't have to find (or create) an API key each time I want to run it.

### Parameters

*Octopus API Key* - This step uses the API to create the tenant so it requires a valid API key with the permissions required for adding tenants.

*Tenant Name* - The name of the tenant to add. Has to be unique for the server otherwise the step will fail.

*Tenant Tags JSON* - A JavaScript array of tenant tags to apply to the new tenant.

Tenant tags aren't represented by Id in API calls. Instead they are represented as _"TagSetName/TagName"_. If I want to apply the Beta tag from the ReleaseGroup set and the SSO tag from the Features set, the parameter value would be _["ReleaseGroup/Beta", "Features/SSO"]_.

*Project and Environments JSON* - A JavaScript object representing which projects and environments to connect the new tenant to.

To set this one up, you'll need to know the project Ids and environment Ids that you want to configure. One way to get this would be to set up a tenant with the desired connections and then to use the API or browser dev tools to peek at the _ProjectEnvironments_ property. Once you have the Ids, set this variable to the JSON that represents the correct mapping. For example, _{ "Projects-1": ["Environments-1", "Environments-2"], "Projects-2": ["Environments-1", "Environments-2", "Environments-3"] }_.

But hey, if you're going to set up a tenant with the desired connections just to get the Ids out of it, might I recommend another step template?

## Clone Tenant

### Functionality

This step template will clone an existing tenant's tags, project connections, and variables (optionally) to a new tenant. This is great if you have a bunch of projects and environments to connect or if you have different categories of tenants (SaaS vs On Premise, etc) that have different tags, projects, or default variables.

### Parameters

*Octopus API Key* - Second verse, same as the first. We need an API key to access and save tenant data.

*Id of Tenant to Clone* - The Id of the tenant to clone (_Tenants-1). The good news is this Id is in the url when you view a tenant. If you set up multiple tenant templates, you can take advantage of prompted variables here too and choose which tenant you want to clone at deploy time.

*New Tenant Name* - The name to use for the new tenant.

*Clone Variables?* - Magic! If this box is checked, the step will also copy the source tenant's variables to the new tenant with the exception of sensitive variables.

## Conclusion

And that's it! The first Better Know a Step Template is in the books, er, blog! Is there a community template that you depend on for your deployments and makes your work easier? Shout it out in the comments and you might see it in the next edition of Better Know a Step Template.

Templates Better Known: 2/316