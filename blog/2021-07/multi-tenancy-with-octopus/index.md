---
title: Better multi-tenancy with Octopus Deploy
description: Learn about the benefits of using the multi-tenancy feature in Octopus Deploy.
author: mark.harrison@octopus.com
visibility: private
published: 2021-07-26
metaImage: blogimage-better-multi-tenancy-with-octopus-2021.png
bannerImage: blogimage-better-multi-tenancy-with-octopus-2021.png
bannerImageAlt: Better multi-tenancy with Octopus Deploy
tags:
 - Product
---

![Better multi-tenancy with Octopus Deploy](blogimage-better-multi-tenancy-with-octopus-2021.png)

Most people who use Octopus use it to deploy projects to one or more environment. For customers who are providing Software as a Service (SaaS) applications, they'll typically need to deploy multiple instances of the application for each of their customers.

The good news is there's a feature thats been around since [Octopus 3.4](https://octopus.com/blog/whats-new-multi-tenant-deployments) designed exactly for these types of deployment, [multi-tenancy](https://octopus.com/docs/deployments/patterns/multi-tenant-deployments).

In this post, I walk through approaches to deploying applications without tenants, and then discuss the benefits of using the multi-tenancy feature.

## Deploying without tenants {#deploying-without-tenants}

There are two main ways we see implemented when deploying multiple instances of the same application for each customer:

- Using [multiple projects](#using-multiple-projects)
- Using [multiple environments](#uing-multiple-environments)

While easy to set up, in either case, this can quickly become overwhelming. It doesn't scale well and can result in duplication.

### Using multiple projects {#using-multiple-projects}

In this scenario, you would configure Octopus with multiple projects, each one representing one of your customers. 

Onboarding a new customer typically requires creating all of the resources in Octopus that is required for a successful deployment for the customer including:

- a new Octopus project and lifecycle 
- a new set of deployment targets
- any customer specific "paid-for" environments

In addition, any common steps across the application's deployment process need to be duplicated in the new project. This is usually things like Manual intervention and notification steps.
 
An obvious benefit of this approach is seeing exactly which release is in which environment, for which customer.

![Multi-tenancy using multiple projects](multiple-projects.png)

This approach allows complete isolation of deployment process for each customer. For example, making a change to one project's process only affects the one customer. You can also tailor the deployment process for the customer depending on what features they have signed up for.

![Multi-tenancy multiple projects customised deployment process](multiple-projects-customised-deployment-process.png)

- Cons: //TODO

### Using multiple environments {#using-multiple-environments}

Another is a single project deployed to many customer environments. 

![Multi-tenancy using multiple customers](multiple-projects.png)

- Pros: //TODO

- Cons: //TODO

## Deploying with tenants {#deploying-with-tenants}

Using Tenants in Octopus allow you to easily create customer specific deployment pipelines without duplicating project configuration. You can manage separate instances of your application in multiple environments in a single Octopus project.

//TODO

## Conclusion {#conclusion}

This post covered some of the common approaches we see when customers deploy projects to one or more environments.

I hope you can see how the Octopus multi-tenancy feature can be leveraged for scalable, reusable, simplified deployments

Happy deployments!

## Learn more {#learn-more}
