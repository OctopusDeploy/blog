---
title: Benefits of Isolated Tenanted Infrastructure
description: A brief summary of the post, 170 characters max including spaces.
author: firstname.surname@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - tag
---

SaaS providers must isolate their customer’s data.  Isolation can be accomplished in code or via isolated infrastructure.  This article will walk you through the pros and cons of each approach and why I recommend isolated infrastructure.

## Isolated Data Requirements

Imagine you provide software to soft-drink manufacturers.  From a legal and business point of view, Coca-Cola wants to ensure Dr. Pepper, PepsiCo, or any other company, cannot access their data.  The same applies to Dr. Pepper, PepsiCo, and every other soft-drink manufacturer.

Outside of losing business, the penalties for failing to comply with that requirement have additional monetary impacts.  Generally, for B2B providers, user agreements or contracts enforce that rule.  Failure to comply could result in fines or lawsuits.  

## Data Isolation Options

Data segregation, or isolation, can be accomplished in multiple ways:
 - A unique identifier, such as CompanyId, is appended to every database record.
 - A database schema (for relational databases) or container/collection (NoSQL databases) per customer.
 - A database per customer.

There are two options for the application layer:
A shared application where everyone logs into the same user interface.
A copy of the application per customer.  Each customer has a unique URL they log into.

How you isolate your customer data impacts the application layer.

|                                     | Shared Application | Isolated Application |
|-------------------------------------|--------------------|----------------------|
|Unique Customer Identifier per record|![](green-check.jpg)|![](red-x.jpg)        |
|Database Schema Per Customer         |![](red-x.jpg)      |![](green-check.jpg)  |
|Database Per Customer                |![](red-x.jpg)      |![](green-check.jpg)  |

**Please Note:** All options marked with a red x are possible but impratical and not recommended.

## Unique customer identifier per record
A unique customer identifier per record, or the shared application and database model, has many benefits.  

- A single codebase to manage.
- All customers get all the new bug fixes and features at once.
- One production database to query.
- Adding a new customer is as simple as adding a record to a table.
- Deployments are simple; push all the application components and database simultaneously.

Despite its many benefits, it does have drawbacks.

### Application Complexity

Adding a unique customer identifier per record is not easy. If the application has existed for years, retrofitting such a change will take 100s, if not 1000s of hours.  It is not just development time; you must include all the time spent testing, writing requirements, analyzing business rules, and more.  

**Picking the customer identifier per record will impact your application for the rest of its life.**

Every new feature, table, configuration, test, and requirements doc must include the customer identifier.  Every new where clause must start “Where CustomerId =.”  A robust series of tests ensures one customer cannot access another customer’s data.

### Noisy customer performance impact

A noisy customer consumes more CPU, RAM, and Database resources than their counterparts.  Going back to the soft-drink example.  Coca-Cola has many brands, while a specialty manufacturer of root beer might have one or two.  When Coca-Cola runs a query to get the sales metrics for all its brands, it will have more data than the specialty manufacturer.  Their query will consume more resources.

About ten years ago, I was a developer on an application that used this model and generated reports for customers.  We didn’t specify a date range for the report.  One customer decided they wanted the last three years of data.  Generating that report consumed so many resources that it prevented everyone else from using the application. 

### Database and Table Size

You will likely have huge tables if your application uses a relational database.  Those tables (and attached indexes) will consume a lot of space, further increasing the database size.  Performing routine tasks such as rebuilding indexes or full backups will take progressively longer and longer.  

### The risk of cross-talk
Cross-talk is when one customer’s data is exposed to another.  There are multiple potential ways cross-talk can occur.  Some examples include:

- A where clause has a hardcoded customer identifier.  Every other customer can see that customer’s data.
- The authentication/authorization mechanism has a bug; a user gets the wrong customer identifier.  They can view and change that customer’s data.
- A malicious user discovers a SQL injection flaw within the application.  They can change their user record to view any customer’s data.

### Customer sign-off and deployments
Every customer will be on the same application version.  That is both beneficial and detrimental.  The benefit is there is one version to support.  It’s detrimental because each customer has a different capacity for change.  Any UI changes, security fixes, bug fixes, or system-wide features.

Returning to the soft-drink example, imagine a security flaw within the application that impacted Coca-Cola and PepsiCo.  The fix is intrusive enough that both customers demand sign-off from their security teams before deploying to Production.  PepsiCo’s security team had the capacity and signed off on the fix within a few days.  Coca-Cola’s security team didn’t have the same capacity, and the sign-off couldn’t happen until next month.

PepsiCo must now wait for Coca-Cola to sign off.  Meanwhile, that issue still exists.  To help PepsiCo, you can add a feature flag turning off the fix for Coca-Cola.  But that requires additional testing.

If you have a customer sign-off process, you must reduce deployment frequency or increase application complexity. 

## Shared application with a database per customer
The logical step to solve application complexity, cross-talk, and database size issues is leveraging a shared application with a database per customer.  In fact, before working at Octopus Deploy, I remember having conversations as a developer and later architect on this topic.

The shared application, database per customer approach, is appealing for many reasons:
- Single code base WITHOUT needing a where clause containing “where customerId=” for every query.
- Can make an existing application “multi-tenant” with minimal code changes.
- The customer’s data is isolated from other customers.  Additional safeguards, such as unique database accounts, help prevent customers from seeing each other’s data.
- Isolated databases lower the risk of a noisy customer causing issues with other customers.
- Every customer is on the same version of the application.  Everyone gets bug and security fixes at the same time.

**Please note:** A schema per customer is nearly identical to this approach.  The primary difference is that it doesn’t solve the database size, performance, and maintenance issues.  

I’ve spent over five years building multi-tenant applications using this approach.  After seeing the good and the bad, I would avoid it at all costs.

### Doesn’t solve all the problems
The shared application with an isolated database model does not solve all the problems found in the shared application and database model.

- Noisy customers can still consume a lot of computing resources.
- Customer sign-off still requires a reduced deployment frequency or increased application complexity.

### Switching connection challenges

The database connection string will be dynamic; it must be retrieved and stored per user.  That is much more complex than storing database connection information in an environment variable or configuration file.

A shared application with an isolated database model requires a “database mapping” layer.  When a user logs in, the application looks up customer connection information in a database, secret vault, or data store.  Only then can the application connect to the correct customer database.

![Diagram on how connections will be managed](single-application-unique-database-per-customer-connection-management.jpg "width=500")

Managing those connection strings becomes more complex and risky because the application source code requires access to the data store.  

### Cross-talk still occurs and is more insidious
The connection information is likely tied to a user’s session in server memory for performance reasons.  With the rise of SOA and microservice-based architecture, customer data exists across multiple applications.  When the application needs to invoke a service on behalf of the user, it’ll send the customer identifier to that service which also has a database per customer.  

This architecture assumes the connection lookup will be perfect 100% of the time.  That’s an impossible standard.  Back-end bugs are complicated to identify.  A user will notice if that connection lookup is incorrect on the front end. A backend error might not be noticed for weeks until someone complains about missing or strange-looking data.

Fixing the data with a shared application and isolated database model requires moving data (and all the child data) between databases and the required clean-up.  While fixing the data with a shared application and database model involves updating a few records.  
Each new customer increases the deployment time
Every customer uses the same application version.  Because of that, the deployment must update all the customers’ databases before deploying the code.  Depending on the complexity of the changes, an application with 50 to 80 customers could require 2-3 hours to update all databases.

### A shared database will likely exist

Not all data is the same.  Some data is static is common across customers.  For example, a list of states and countries.  I’ve seen applications with a shared database to store that common data to avoid synchronizing to all the customer databases.  Each new feature and bug fix must know which data store to use.

## Isolated tenant infrastructure
The isolated tenant infrastructure solves all the above problems:

- No risk of cross-talk; a customer’s copy of the application can only communicate with the customer’s database.
- Noisy neighbors are controlled via CPU, RAM, and Database resource limits.  When they consume all their resources, they don’t impact other customers.
- No complex connection string mapping.  An environment variable or configuration file stores the connection string.
- Simpler code base, as is no “where customer=” logic permeates throughout the code.
- Sign-off is much easier.  Upgrade the customers signed off on a bug fix, new feature, or security enhancement.  Leave the other customers alone until sign-off.
- Performance and latency improvements by running the application closer to the customer’s primary users in a specific region or country.  
- Easier to support multiple application versions.  
- Deployments can be scheduled for each customer’s “off-hours.”

The isolated tenant infrastructure is not without its faults.

### It takes more time to add a customer
With the shared application and database model, adding a customer was as simple as adding a row to a table.  With isolated tenant infrastructure, customers must wait minutes or hours for their infrastructure.  

Infrastructure as Code and automation is required for the isolated tenant infrastructure to scale.

### More deployments are required

A single deployment is needed with the shared application and database, and shared application and database models.  A deployment could take a single night.  

With isolated tenant infrastructure, each customer will require a deployment.  It could take multiple days to deploy a change to all customers. A deployment automation tool is required to manage all these deployments.

### More versions to support
 
The likelihood of all customers being on the same version is low.  One group of customers might be on 2023.3.568, while another group is on 2023.2.990.  Most SaaS products have different tiers or features available as add-ons.  

The possible versions and feature sets to support will exponentially grow.  That requires robust testing automation, including the ability to spin up new “test customers” or “test environments” to support all the various permutations.  

### Higher Cloud Bills
With a shared application and database model, all customers share the same pool of resources.  The chances of all customers needing all the resources at any given time are near zero.

However, the inverse is true with isolated tenanted infrastructure.  Applications that can only run on Virtual Machines will “bursty” resource use.  A lot of time, the VM will be idle, consuming minimal resources.  As such, they will incur higher cloud bills than those that run on platforms that allow sharing of resources with the required isolation.  For example, container platforms such as ACS, ECS, Kubernetes, PaaS platforms such as Azure Web Apps, Lambdas, and Functions, or Azure’s Elastic database pools. 

That said, the additional cloud bills will likely be significantly cheaper than the people-hours to manage code complexity found in the shared application and database model.

## Conclusion

Unless there is a compelling business reason, my default recommendation is isolated tenanted infrastructure.

Many risks are associated with customers sharing the same application and database.  Almost all those risks can be mitigated by leveraging isolated tenanted infrastructure.  Until recently, that approach was considered too costly and complex to manage at scale.  With today's tools and technologies, isolated tenanted infrastructure at scale is achievable.  

Happy deployments!
