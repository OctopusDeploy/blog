---
title: Benefits of isolated tenanted infrastructure
description: A brief summary of the post, 170 characters max including spaces.
author: bob.walker@octopus.com
visibility: private
published: 2023-08-23-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - DevOps
  - Multi-Tenancy
---

SaaS providers must isolate their customer’s data.  Isolation is possible in code or via isolated infrastructure.  This article will walk you through the pros and cons of each approach.  You will see why I recommend isolated infrastructure.

## Isolated data requirements

Imagine you provide software to soft-drink manufacturers.  Coca-Cola doesn't want Dr. Pepper, or PepsiCo, accessing their data.  

Outside of losing business, failing to have data isolation have monetary impacts.  Generally, for B2B providers, user agreements or contracts enforce that rule.  Failure to comply could result in fines or lawsuits.  

## Data isolation options

There are four approaches for data isolation:

 - A unique customer identifier on every database record.
 - Tenant-specific tables in the database.
 - A database schema or container/collection (NoSQL databases) per customer.
 - A database per customer.

There are two options for the application layer:
- A shared application where everyone logs into the same user interface.
- A copy of the application per customer with a unique login URL.  

How you isolate your customer data impacts the application layer.

|                                      | Shared Application    |Isolated Application  |
|--------------------------------------|-----------------------|----------------------|
|Unique Customer Identifier per record |![](green-check.jpg)   |![](red-x.jpg)        |
|Tenant-specific tables in the database|![](green-check.jpg)   |![](yellow-circle.jpg)|
|Database Schema Per Customer          |![](yellow-circle.jpg) |![](green-check.jpg)  |
|Database Per Customer                 |![](red-x.jpg)         |![](green-check.jpg)  |

**Please Note:** All options marked with a yellow circle and red x are possible but impractical and not recommended.

Combining the data isolation with the application yields three options:

- Shared application and database
- Shared application, isolated database per customer
- Isolated tenanted infrastructure

## Shared application and database

The shared application and database is when all customers are stored in the same database and access the same application version.  Commonly, a shared application is paired with the unique customer identifier per record or tenant-specific tables.  That model has many benefits.  

- A single codebase to manage.
- All customers get all the new bug fixes and features at once.
- One production database to query.
- Adding a new customer is as simple as adding a record to a table.
- Deployments are simple; push all the application components and database simultaneously.

Despite its many benefits, it does have drawbacks.

### Application complexity

Adding a unique customer identifier per record is not easy. If the application has existed for years, retrofitting such a change will take 100s, if not 1000s of hours.  Every query must start with `Where CustomerId =`. It is not only development time.  Include all the time spent testing, writing requirements, analyzing business rules, and more.  

Moving all customers to tenant-specific tables makes it easier to know when to use `Where CustomerId = ` in the where clause.  But that doesn't remove the requirements.  Opting for a set of tables per customer will remove the `Where CustomerId =` requirement.  However, every query becomes dynamic, as table names must change per customer when the query is being built.  

**Picking the customer identifier per record will impact your application for the rest of its life.**

How you develop features will change.  Is the feature global, such as a sign-in flow?  Or is the feature specific to the customer?  A robust series of tests ensures one customer cannot access another customer’s data.

### Noisy customer performance impact

A noisy customer consumes more CPU, RAM, and Database resources than their counterparts.  Going back to the soft-drink example.  Coca-Cola has many brands, while a specialty manufacturer of root beer might have one or two.  Coca-Cola will consume more resources for its queries than the specialty manufacturer.  

I was a developer on an application that used this model about ten years ago.  One feature generated reports for customers.  We didn’t specify a date range for the report.  One customer decided they wanted the three years of data.  While that report ran, every other user could not use the application.
 
### Database and table size

You will likely have huge tables if your application uses a relational database.  Those tables (and attached indexes) will consume a lot of space.  That will increase the database size.  Routine maintenance tasks such as rebuilding indexes or full backups will take longer.

### The risk of cross-talk

Cross-talk is when one customer’s data is exposed to another.  There are potential ways cross-talk can occur.  Some examples include:

- A where clause has a hardcoded customer identifier.  Every other customer can see that customer’s data.
- The authentication/authorization mechanism has a bug; a user gets the wrong customer identifier.  They can view and change that customer’s data.
- A malicious user discovers a SQL injection flaw within the application.  They can change their user record to view any customer’s data.

### Customer sign-off and deployments

Every customer will be on the same application version.  That is both beneficial and detrimental.  The benefit is there is one version to support.  It’s detrimental because each customer has a different capacity for change.  Any UI changes, security fixes, bug fixes, or system-wide features.

Returning to the soft-drink example, imagine a security flaw that harmed Coca-Cola and PepsiCo.  Both customers demand sign-off from their security teams before deploying to Production.  PepsiCo’s security team had the capacity and signed off on the fix within a few days.  Coca-Cola’s security team didn’t have the same capacity, and the sign-off couldn’t happen until next month.

PepsiCo must now wait for Coca-Cola to sign off.  Meanwhile, that issue still exists.  To help PepsiCo, you can add a feature flag turning off the fix for Coca-Cola.  But that requires more testing.

If you have a customer sign-off process, you must reduce deployment frequency or increase application complexity. 

## Shared application with a database per customer
The logical step to solve application complexity, cross-talk, and database size issues is leveraging a shared application with a database per customer.  Looking at the table from above, this section represents the yellow circles.

Before working at Octopus Deploy, I worked on three enterprise-level applications at three companies who adopted this approach.  I won't lie, the shared application, database per customer approach, is appealing for many reasons:

- Single code base WITHOUT needing a where clause containing "where customerId=" for every query.
- Can make an existing application "multi-tenant" with minimal code changes.
- The customer’s data is isolated from other customers.  Unique database accounts help prevent customers from seeing each other’s data.
- Isolated databases lower the risk of a noisy customer causing issues with other customers.
- Every customer is on the same version of the application.  

**Please note:** A schema per customer is nearly identical to this approach.  The primary difference is that it doesn’t solve the database size, performance, and maintenance issues.  All the issues discussed in this section apply to that approach.

I’ve spent over five years building multi-tenant applications using this approach.  After seeing the good and the bad, I would avoid it at all costs.

### Doesn’t solve all the problems

The shared application with an isolated database model does not solve all the problems found in the shared application and database model.

- Noisy customers can still consume a lot of computing resources.
- Customer sign-off still requires a reduced deployment frequency or increased application complexity.

### Switching connection challenges

The database connection string will be dynamic; it must be retrieved and stored per user.  That is much more complex than storing database connection information in an environment variable or configuration file.

A shared application with an isolated database model requires a "database mapping" layer.  When a user logs in, the application looks up customer connection information in a database, secret vault, or data store.  Only then can the application connect to the correct customer database.

![Diagram on how connections will be managed](single-application-unique-database-per-customer-connection-management.jpg "width=500")

Managing those connection strings becomes more complex and risky because the application source code requires access to the data store.  With the other approaches, those values are retrieved by the deployment tool during a deployment.  

### Cross-talk still occurs and is more insidious

The connection information is likely tied to a user's session in server memory for performance reasons.  With the rise of SOA and microservice-based architecture, customer data exists across multiple applications.  When the application needs to invoke a service on behalf of the user, it’ll send the customer identifier to that service which also has a database per customer.  

This architecture assumes the connection lookup will be perfect 100% of the time.  That’s an impossible standard.  Back-end bugs are complicated to identify.  A user will notice if that connection lookup is incorrect on the front end. A backend error might not be noticed for weeks until someone complains about missing or strange-looking data.

Fixing the data with a shared application and isolated database model requires moving data (and all the child data) between databases and the required clean-up.  While fixing the data with a shared application and database model involves updating a few records.  It is very risky to generate a script per incident.  Development teams will create custom tooling enforcing business rules to mitigate risk.

### Each new customer increases the deployment time

Every customer uses the same application version.  Because of that, the deployment must update all the customers’ databases before deploying the code.  Depending on the complexity of the changes, an application with 50 to 80 customers could require 2-3 hours to update all databases.

### A shared database will likely exist

Not all data is the same.  Some data is static is common across customers.  For example, a list of states and countries.  I’ve seen applications with a shared database to store that common data to avoid synchronizing to all the customer databases.  Each new feature and bug fix must know which data store to use.

## Isolated tenant infrastructure

The isolated tenant infrastructure solves all the above problems:

- No risk of cross-talk; a customer’s copy of the application can only communicate with the customer’s database.
- Noisy neighbors are controlled via CPU, RAM, and Database resource limits.  When they consume all their resources, they don’t impact other customers.
- No complex connection string mapping.  An environment variable or configuration file stores the connection string.
- Simpler code base, as is no "where customer=" logic permeates throughout the code.
- Sign-off is much easier.  Upgrade the customers signed off on a bug fix, new feature, or security enhancement.  Leave the other customers alone until sign-off.
- Performance and latency improvements by running the application closer to the customer’s primary users in a specific region or country.  
- Easier to support multiple application versions.  
- Deployments can be scheduled for each customer’s "off-hours."

The isolated tenant infrastructure is not without its faults.

### It takes more time to add a customer

With the shared application and database model, adding a customer was as simple as adding a row to a table.  With isolated tenant infrastructure, customers must wait minutes or hours for their infrastructure.  

Infrastructure as Code and automation is required for the isolated tenant infrastructure to scale.

### More deployments

Only a single deployment is needed with the shared application and database, and shared application and database models.  A deployment could take a single night.  

With isolated tenant infrastructure, each customer will require a deployment.  It could take multiple days to deploy a change to all customers. A deployment automation tool is required to manage all these deployments.

### More versions to support
 
The likelihood of all customers being on the same version is low.  One group of customers might be on 2023.3.568, while another group is on 2023.2.990.  Most SaaS products have different tiers or features available as add-ons.  

The possible versions and feature sets to support will exponentially grow.  That requires robust testing automation, including the ability to spin up new "test customers" or "test environments" to support all the various permutations.  

### Higher cloud bills

With a shared application and database model, all customers share the same pool of resources.  The chances of all customers needing all the resources at any given time are near zero.

However, the inverse is true with isolated tenanted infrastructure.  Applications that can only run on Virtual Machines will have "bursty" resource use.  A lot of time, the VM will be idle, consuming minimal resources.  As such, they will incur higher cloud bills than those that run on platforms that allow sharing of resources with the required isolation.  For example, container platforms such as ACS, ECS, Kubernetes, PaaS platforms such as Azure Web Apps, Lambdas, and Functions, or Azure’s Elastic database pools. 

That said, the cloud bills will likely be cheaper than the people-hours to manage code complexity found in the shared application and database model.

## Conclusion

Unless there is a compelling business reason, my default recommendation is isolated tenanted infrastructure.

Many risks are associated with customers sharing the same application and database.  Almost all those risks can be mitigated by leveraging isolated tenanted infrastructure.  Until recently, that approach was considered too costly and complex to manage at scale.  With today's tools and technologies, isolated tenanted infrastructure at scale is achievable.  

Happy deployments!