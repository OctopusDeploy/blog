---
title: Connecting a project to multiple tenants as a bulk action
description: Learn about our new bulk actions for tenanted deployments in Octopus. Connect a project to multiple tenants and add or clone a new tenant straight from the project page.
author: ian.khor@octopus.com
visibility: private
published: 2023-09-20-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - Product
  - Multi-Tenancy
---

Tenants in Octopus have become the standard way to create customer-specific deployment pipelines without duplicating project configurations. It's best used to deploy multiple instances of an application for each of your customers. 

We had feedback that people found adding or connecting a project to multiple tenants difficult. This happens quite frequently for some of our largest Tenants users. For example, if you have a batch of new customers/clients/machines you need to connect to an existing project. Another example is when you have a new project (which represents a new application, product, or microservice) that you need to attach to thousands of tenants. 

We addressed this with a new bulk action. You can now connect a project to multiple tenants as a bulk action in the Octopus user interface (UI).

The new bulk action will let you connect some or all of the tenants in a particular space to a new or existing project. You do this via a wizard-style format, where we guide you through the elements you need to connect (for example, tenants and environments). After the connection operation starts, you'll see a progress bar/counter, to help you track how many tenants are being connected to the project and when the bulk action operation finishes.

![An overview of the new bulk action]

In this post, I show you how to connect a project to multiple tenants with the new bulk action.

## The new Tenants page in the Project view

You can see the new tenants page in the Project view below.

![The new tenants page in the project view]

To access the new Tenants page, go to Projects and then the sidebar menu.This page provides:

- A full list of all the tenants connected to this project
- A full list of the environments connected to each of the tenants in this project

You'll find 3 new options in the top right-hand corner of the page. One of them is the bulk action, which you can use by clicking on the green Connect Tenants button.  The other 2 functions let you add a new blank tenant, and clone an existing tenant.

![Creating a new tenant from the new tenants page in a project]
![Cloning an existing tenant from the new tenants page in a project]

:::hint 
Although the add and clone functionality exists in the project, any new or cloned tenants *do not* get automatically connected to the project you created or cloned the tenant from.
:::

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
