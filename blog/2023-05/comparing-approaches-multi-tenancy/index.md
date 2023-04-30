---
title: Comparing approaches to multi-tenancy
description: TBC
author: steve.fenton@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 
isFeatured: false
tags: 
  - tag
---

Introduction...

## The problems with organic multi-tenancy

The organic approach to multi-tenancy is also the worst. You might recognize this picture. After landing a sale to one big customer, you develop an application to solve some problem they have. The contract allows you to retain the IP and sell the software you create to other customers, but for some time, there's just the one big client.

After building a compelling feature set, you find yourself with your second customer. They aren't as big as the first, but it gets the ball rolling on your sales and on-boarding process. The only real problem is this new customer is set up a little differently, so you need to make a few tweaks here and there.

Before you know it, you have two or three versions of you application with minor differences, powering each of your customers. You don't notice any problems until the following starts to happen:

1. When you fix a bug, you have to repeat the fix in multiple branches or source code repositories
2. Sometimes, you can't apply the same bug fix as the code is different for some of the customers
3. You find that two customer have a similar customization, but the database column has a different name for each customer
4. Your deployments used to take an hour when you had one customer, with 5 customers it takes a whole day and often goes wrong
5. It takes longer to test the software than it does to change it
6. You have a process for new customers where you try to identify the codebase that is closest to what they need from your list of codebases.

With organic multi-tenancy, the more successful you are the harder everything gets. 

## Approaches to multi-tenancy

Multi tenancy approaches are plotted using two measures. The number of databases and the number of running applications. 

It should never be plotted along a scale for the number of codebases or the number of data schemas like the organic multi-tenancy approach. You should maintain one codebase and one schema and design flexibility where it makes viable sense to the product. You can lower the cost of flexibility using techniques such as feature flags, but flexibility always costs you something. It shouldn't cost you the maintainance of multiple codebases or schemas for the same application.

:::hint
To avoid confusion, an installation is a deployment of one application. If you load balance the application, you may have several instances of the same deployment. This is still one installation for the purposes of this discussion.
:::

1. One data store, one application installation
2. Data store per tenant, one application installation
3. One data store, application installation per tenant
4. Data store per tenant, application installation per tenant

![The four quadrants of multi-tenancy, based on application installations and data stores](multi-tenancy.png)

These are the four groups for multi-tenancy approaches.

### Multi-tenancy considerations

- Hard walls for data
- Data residency
- Noisy neighbours
- Tracking or limiting customer usage

### One data store, one application installation

At the heard of the single data store and application installation is a tenant concept. You'll have a table in your data store called *tenants* and each customer is listed with a unique identity. Each tenant represents a virtual wall around a set of data.

Almost all information in the data store is linked back to a tenant. A common exception is that users mare typically global, and granted permission to one or more tenants.

### Data store per tenant, one application installation

In this scenario, your application is data store aware. There is some signal built into the application that tells it which data store to connect to. For example, you may offer subdomains that map to each data store so `octo-pet-shop.example.com` connects to an OctoPetShop data store and `random-quotes.example.com` connects to a RandomQuotes data store.

Each data store would have the same schema, but the data would be tenant-specific. Schema updates would need to be deployed to all data stores, and your backup and maintenancy strategy would also need to be applied uniformly.

### One data store, application installation per tenant

Having one data store and an application installation per tenant is useful if the application itself is resource intensive, but the database isn't. Each tenant may pay for a different tier of compute resource, based on their intended workflows.

Each application installation has the same data store connection, but its own allocated compute resources. This is very similar to the full muti-tenanted application except you install the application for each tenant to give them dedicated compute and to limit their usage.

### Data store per tenant, application installation per tenant

The infrastructure for this scenario is similar to that required for the organic multi-tenancy. Each customer has their own dedicated application and data store, usually on isolated resources.

The crucial difference compared to organic multi-tenancy is that there is one schema definition and one application codebase.

This scenario has been made more managemeble thanks to virtualization and containerization infrastructure technology.

## Mixed multi-tenancy approaches

It is also possible to use a mixed approach where you:

1. Use one data store and one application installation for groups of small customers
2. A data store and application installation for a single larger enterprise customer

The mixed approach has some benefits. The multi-tenant installation for small customers means resource usage is shared efficiently by many users. For the large enterprises, their dedicated installation and data store means they don't become noisy neighbours to other customers and their data has thick hard walls from other customers. The enterprise customers may even have data residency requirements that mean their installation is within a specific region or their own data centre.


## How they compare as you scale

One customer

All approaches are similar, though any multi-tenancy added at this stage will be an investment in your future (i.e. additional work that won't pay dividends until you land your second customer).

### Data store pros and cons

Single data store
- If one tenant accidentally deletes data, how do you resore their data without affecting other tenants
- You must ensure multi-tenancy is robust and data cannot leak
- Maintenance windows are harder to schedule if you have customers in many time zones
- A heavy database operation or rogue report can block requests for all tenants

Tenant data store
- Easy to manage the data in the store as it all belongs to one customer
- Harder to manage maintenance plans and backup tasks

### Application installation pros and cons

Single application installation
- Resources are shared, which means you need fewer CPUs and other resources
- Load balancing can smooth usage across a single installation
- Redundancy requirements are lower when you have more than 3 customers (n+3 for multi-tenant vs n*2 for each customer)
- Noisy neighbours can cause slowdowns for other customers
- Many customers in a single region can cause large peaks at a specific time

Application installation per tenant
- Easier to track and limit usage per customer
- Can be installed in customer-selected regions and data centres

### Virtualization and containers

Many of the drawbacks to tenant-specific installations have become less concerning thanks to virtualization and containerization.

Virtual machines provided an easy way to quickly add customer-specific resources and containers have made that even easier, especially with the automation and config-as-code technologies that are now available.

Creating a set of containers specific to a customer can now be completed as an automated step as part of your on-boarding process.

## What to do when you don't know the future

It is worth addressing the question of multi-tenancy early as this is one of the few truly architectural decisions you can make. Your chosen path for multi-tenancy can heavily impact many decisions that will be made as your write the application and these are hard to change later.

You should either:

- Add a tenant concept early, such as a tenant identity, to keep your options open, or
- Invest heavily in isolated customer instances, including automation of customer specific installations

Having multi-tenancy available from the start may benefit your testing as people can have their own test tenant and avoid colliding with other tests. Having an "integration test" tenant with a well-designed and scripted dataset will help you maintain reliable integration tests and UI test suites.

## Conclusion

Happy deployments!
