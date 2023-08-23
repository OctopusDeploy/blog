---
title: Connecting a project to multiple tenants as a bulk action
description: A blog post that introduces a new bulk action for Octopus multi-tenancy users. This new feature will allow users to connect a project to multiple tenants using a UI-based wizard flow. Additionally, we will also be allowing users to add or clone a new tenant straight from the project page itself.
author: ian.khor@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - Product
  - Multi-Tenancy
---

See https://github.com/OctopusDeploy/blog/blob/master/tags.txt for a comprehensive list of tags,

Tenants in Octopus have become the standard way for our customers to create customer specific deployment pipelines without duplicating project configurations. It is best used to deploy multiple instances of an application for each of our customers. Customers found it difficult to add or connect a project to multiple tenants, which can happen quite frequently for some of our largest multi-tenancy users. For example, where customers either have a batch of new customers/clients/machines that need to be connected up to an existing project, or there is a new project (which represents a new application, product or microservice) that needs to be attached to 1000s of the appropriate tenants when necessary.

We are pleased to have addressed this by allowing customers to connect a project to multiple tenants as a bulk action in the Octopus UI .

The new bulk action will allow customers to connect some or all of the tenants within a particular space and connect them to a new or existing project. This is facilitated by a wizard style format, where we walk you through the different elements that need to be connected (e.g. tenants and environments) in a guided manner. Once the connection operation commences, customers are also provided with a progress bar/counter, that helps them keep track of how many tenants are being connected up to the project and when the bulk action operation finally ends.

![An overview of the new bulk action]

In this post, I'll show you how the new bulk action of connecting a project to multiple tenants work

## The new Tenants page in the Project view

The new tenants page in the Project view can be seen below

![The new tenants page in the project view]

As can be seen in the screenshot above, the new tenants page can be accessed via the sidebar menu after clicking on the top-nav ‘Projects’ page. This page will provide:

A full list of all the tenants that are connected to this project; and
A full list of the environments that are connected to each of the tenants in this project.

The top right hand corner of the page will have three new options. One of them is the bulk action, which can be initiated by clicking on the green “Connect Tenants” button. The other two functions is the ability to add a new blank tenant, as well as the ability to clone an existing tenant.

![Creating a new tenant from the new tenants page within a project]
![Cloning an existing tenant from the new tenants page within a project]

**NOTE**: Although the add and clone functionality exists within the project, any new or cloned tenants will not automatically be connected to the project from which the tenant was created or cloned from.

## The wizard style format of attaching tenants to a project

![The wizard style format of attaching tenants to a project]

The method of attaching multiple tenants to a project follows a step by step, wizard style workflow. This takes the user from one important step to the next (e.g. selecting all the tenants, then selecting all of the environments that need to be attached to that tenant) simply and efficiently. This also provides sufficient guardrails and instruction to the user so that not only are they capable and familiar enough with the process to complete it themselves but that, at completion, they can be confident and comfortable that they have followed the process correctly when attaching 1000s of tenants to a project.

## The ability to select some or all of the tenants to be connected to the project

![Selecting some or all of the tenants to be connected to a project]

This bulk action will give users the ability to either select some of the tenants that they have created in this space to be attached to this project. It will also give users te ability to select all of the tenants in that given space if they want to attached all of them to the single project.

This gives users flexibility in terms of maybe selecting only some of the tenants as test tenants, which is important when we consider the next step which is attaching environments to that tenant, or selecting all of them if all of them have the same set of environments, such as all tenants being production tenants for example.

## The ability to assign some or all of the environments to the tenants in the project

![Assigning some or all of the environments to the tenants in the project]

Assigning environments to the tenants happens in bulk as well. The user can either select individual environments that they would like to attach to a tenant, or they also have the option of assigning all of the available environments in the project to all of the tenants that were selected during the previous wizard step.

This flexibility in environment selection is provided for 2 reasons:
- Gives the user the ability to create 1 environment to 1 tenant connections if they want to in bulk, especially if they are creating a batch of test or production tenants in 1 go; and
- Gives the user the ability to connect a whole suite of environments to the tenants in cases where they do not respect the 1 environment to 1 tenant rule (which happens alot!)

## A way to indicate completion of the connection

![Connection progress bar]

Once all of the tenants are selected and the appropriate environments are connected, the user can hit the ‘Connect’ button. The bulk connection operation will then commence. A progress indicator will appear on the tenant scree, which will not only show how many tenants are being connected to the project, but also how many more are required to be connected before the connection is completed.

The progress message will be shown for any user that accesses the tenants page during the connection cycle. Once the connection operation is complete, the message will show success and will be dismissable by any user as well.

## Conclusion
The new bulk action allows you to connect all of your relevant tenants to new or existing projects in a simple, efficient and intuitive manner.

We'd love feedback on this feature while we continue to refine it. If you're an Octopus Cloud customer, it's available now from 2023.3.11654 and above.

<span><a class="btn btn-success" href="https://octopusdeploy.typeform.com/to/iBkrLS52">Share your feedback</a></span>

Happy Deployments!
