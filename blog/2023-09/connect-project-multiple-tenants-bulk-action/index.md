---
title: Connecting a project to multiple tenants as a bulk action
description: Learn about our new bulk actions for tenanted deployments in Octopus. Connect a project to multiple tenants and add or clone a new tenant straight from the project page.
author: ian.khor@octopus.com
visibility: public
published: 2023-09-25-1400
metaImage: blogimage-multitenancy-bulkactions-2023.png
bannerImage: blogimage-multitenancy-bulkactions-2023.png
bannerImageAlt: Octopus server containing multiple cube shaped check boxes to represent batch automation. Lines are connected to the server and attached to infinity bikes representing tenants and delivery.
isFeatured: false
tags: 
  - Product
  - Multi-Tenancy
---

Tenants in Octopus have become the standard way to create customer-specific deployment pipelines without duplicating project configurations. It's best used to deploy multiple instances of an application for each of your customers. 

We had feedback that people found it difficult adding or connecting a project to multiple tenants. This happens quite frequently for some of our largest Tenants users. For example, if you have a batch of new customers, clients, or machines you need to connect to an existing project. Another example is when you have a new project (which represents a new application, product, or microservice) that you need to attach to thousands of tenants. 

We addressed this with a new bulk action. You can now connect a project to multiple tenants as a bulk action in the Octopus user interface (UI).

The new bulk action lets you connect some or all of the tenants in a particular space to a new or existing project. You do this using a wizard-style workflow, where we guide you through the elements you need to connect (for example, tenants and environments). After the connection operation starts, you see a progress bar/counter, to help you track how many tenants are being connected to the project and when the bulk action operation finishes.

![An overview of the new bulk action](blogimage-bulkactionwizardwindow.png)

In this post, I show you how to connect a project to multiple tenants with the new bulk action.

## The new Tenants page in the Project view

You can see the new tenants page in the project view below.

![The new tenants page in the project view](blogimage-newtenantspage.png)

To access the new Tenants page, go to **Projects** and then the sidebar menu. This page provides a full list of:

- All the tenants connected to this project
- The environments connected to each of the tenants in this project

You'll find 3 new options in the top right-hand corner of the page. One of them is the bulk action, which you can use by clicking on the green **Connect Tenants** button.  The other 2 functions let you add a new blank tenant, and clone an existing tenant.

![Creating a new tenant from the new tenants page in a project](blogimage-addtenant2.gif)

![Cloning an existing tenant from the new tenants page in a project](blogimage-clonetenant2.gif)

:::hint 
Although the add and clone functionality exists in the project, any new or cloned tenants *do not* get automatically connected to the project you created or cloned the tenant from.
:::

## The wizard-style format of attaching tenants to a project

![The wizard style format of attaching tenants to a project](blogimage-tenantconnectwizardoverview.gif)

Attaching multiple tenants to a project follows a step-by-step, wizard-style workflow. This takes you from one important step to the next. For example, selecting all the tenants, and then selecting all the environments you need to attach to that tenant. 

This provides guardrails and instructions so you become familiar with the process. At completion, you can be confident you followed the process correctly when attaching thousands of tenants to a project.

## The ability to select some or all of the tenants to be connected to the project

![Selecting some or all of the tenants to be connected to a project](blogimage-selecttenantsconnect.gif)

This bulk action lets you select:

- Some of the tenants you created in this space to attach to this project. 
- All the tenants in that given space if you want to attach all of them to the single project

This gives you the flexibility to select only some of the tenants as test tenants. This is important when considering the next step - attaching environments to the tenant or selecting all, if all have the same set of environments. For example, when all tenants are production tenants.

## The ability to assign some or all of the environments to the tenants in the project

![Assigning some or all of the environments to the tenants in the project](blogimage-selectenvironments.gif)

You can assign environments to the tenants in bulk too. You can:

- Select individual environments to attach to a tenant
- Assign all available environments in the project to all the tenants you selected during the previous wizard step

We provide this flexibility in environment selection for 2 reasons:

- So you can create one environment to one tenant connections in bulk, especially if you're creating a batch of test or production tenants in one go
- So you can connect a whole suite of environments to the tenants when they do not follow the one environment to one tenant rule

## Completion of the connection

![Connection progress bar](blogimage-progressbar.gif)

After you select all the tenants and connect the appropriate environments, you can click the **Connect** button. The bulk connection operation then starts.

A progress indicator appears on the Tenants screen. This shows how many tenants are being connected to the project, and how many more still need to connect before the process is complete.

Anyone accessing the Tenants page during the connection cycle will see the progress message. After the connection operation is complete, you see a success message that anyone can dismiss.

## Conclusion

The new bulk action lets you connect tenants to new or existing projects in a simple, efficient, and intuitive way.

We'd love feedback on this feature while we continue to refine it. If you're an Octopus Cloud customer, it's available from versions 2023.3.11654 and above. For self-hosted customers, it's available in the latest 2023.3 server version.

<span><a class="btn btn-success" href="https://octopusdeploy.typeform.com/to/iBkrLS52">Share your feedback</a></span>

Happy deployments!
