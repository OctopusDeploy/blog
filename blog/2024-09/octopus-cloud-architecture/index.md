---
title: Octopus Cloud Architecture
description: Learn how cell-based architecture helped us make Octopus Cloud reliable and scalable.
author: pawel.pabich@octopus.com
visibility: public
published: 2024-09-02-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - Engineering
  - Octopus Cloud
  - Azure
---

Octopus Cloud runs reliably thousands of Octopus Server instances. In this post, I explain the architecture that underpins this complex distributed system.

## What is Octopus Cloud

Octopus Cloud is the easiest way to run Octopus Deploy. It has the same software and functionality as Octopus Server, except we host it for you. An Octopus Server instance hosted in Octopus Cloud is called a Cloud instance. You don’t need to download, install, and manage it yourself. Check out [our documentation](https://octopus.com/docs/octopus-cloud) if want to give it a try.

## Foundations of Octopus Cloud

We've designed Octopus Cloud to be a distributed system that can withstand partial infrastructure outages. To achieve this high-level goal we've broken it down into a set of guiding principles, which the engineering team at Octopus has been following since the inception of the product.

### Avoid single point of failure

Octopus Cloud is based on what’s sometimes called a cell-based architecture. A cell represents an independent and isolated part of a larger system that can be replicated to increase the system's scalability and reliability. Individual cells can be created, destroyed, upgraded, scaled, or simply stay broken without affecting the rest of the system.

![Different types of cells in Octopus Cloud](cells.png)

#### Azure region as a cell

Octopus Cloud is currently deployed in [3 different Azure regions](https://octopus.com/docs/octopus-cloud#octopus-cloud-hosting-locations) and can be easily deployed to more regions if we see enough demand there. Resources used by Octopus Cloud in each Azure region are fully isolated from each other. This means that an outage in `West US 2` region will only affect customers hosted in that region, and won't affect customers hosted in `West Europe` and `Australia East` regions.

#### Reef as a cell

Resources deployed into each Azure region also follow the cell-based architecture. There is more than one reef in each region. A reef is a collection of Azure resources shared by a number of Cloud instances. Reefs don’t share any resources, so an outage of one of the reefs in a region doesn’t affect all other reefs in that region.  

Fun fact. We've picked `Reef` as the name because this is where octopuses usually live in tropical waters. The other popular contender was Octopus Cloud region, but we felt that it might get easily confused with Azure region.

#### Cloud instance as a cell

Octopus Server is a monolithic .NET application that requires 3 basic resources to run:
* SQL Server database for structured data (e.g. project data)
* Shared file system for unstructured data (e.g. task logs, packages)
* VM/container for computing

Octopus Cloud provides Cloud instances with an Azure version of each of the required resources:

![Mapping of Octopus Server resource requirements to Azure resources provided by Octopus Cloud](resources.png)

This is where the strict isolation driven by the cell-based approach used at the region and reef level changes from a binary proposition to a wide range of options. 

At one end of the spectrum, we can host a single instance on its own dedicated reef. This is a very powerful option, but we avoid using it because it’s not cost-effective.

At the other end of the spectrum, hundreds of instances can run on a single reef and share a single Azure SQL elastic pool, a single AKS cluster, and a single Azure storage account. This option works well for Cloud instances that are active at different times of the day and don’t generate a lot of load.

Finally, we have a few options in between these 2 extremes. For example, a busy Cloud instance will have dedicated resources assigned to its database but more than likely it will share its K8s cluster and storage account with other instances.
 
Each Cloud instance has its own database and a file share, which allows us to use built-in Azure tools for managing backup and restore processes.

### Design for gradual rollouts

The cell-based architecture lends itself to gradual rollouts because the infrastructure in each reef and region can be upgraded independently. This reduces the risk of a change causing an outage that affects the whole of Octopus Cloud. As much as nobody likes outages, an outage that affects hundreds of instances is a better place to be than an outage that affects thousands of instances. 

The decision to provision a dedicated file share and a database for each Cloud instance also helps with the gradual rollout of the Octopus Server changes because each instance can be upgraded and downgraded in isolation. That wouldn't be an easy goal to achieve if, for example, more than one Cloud instance shared the same database via database schemas or a similar mechanism that implements multi-tenancy.

### Scalable by default

This is again where cell-based architecture shines. We can scale the infrastructure of Octopus Cloud easily by adding new reefs. As a side note, we can also assign individual resource limits to each Cloud instance.

Fun fact. We had an event a few years ago when Octopus Cloud doubled the number of active Cloud instances within 48-72 hours, and nothing broke. No existing customers were affected and Octopus simply paid a higher hosting bill that month.

### Avoid temporal coupling

Octopus Cloud must be able to perform its primary function, which is keeping Cloud instances up and running, even when some of the non-essential services aren’t available. For example, Cloud instances should continue to operate when:

* Octopus Cloud monitoring platform is not available
* Octopus Cloud management platform is not available
* Octopus Cloud observability platform is not available

In the context of [CAP theorem](https://en.wikipedia.org/wiki/CAP_theorem), Octopus Cloud opts for availability over consistency at the infrastructure level, and consistency over availability at the individual Cloud instance level.

### Use officially supported APIs only

Octopus Cloud glues together several 3rd party services using their APIs. This might sound like an obvious choice. That said, we’ve extended this approach also to interactions with internal services and Octopus Server itself. For example, if Octopus Cloud needs to provide a configuration value to a Cloud instance for a configuration setting that isn't exposed via a public API, then we will add a new API and won't modify this setting directly in the database.

Fun fact: After years of using APIs from major service and Cloud providers, we've learnt that very few of them follow [SemVer](https://semver.org/), and changes to runtime characteristics (e.g., a synchronous operation becomes asynchronous and vice versa) happen all the time and without any notice. 

## Developer productivity

Cell-based architecture can also increase engineering productivity which is often overlooked. Each change can be tested in each own, isolated reef, either locally or in an automated build. There is no need for shared `Test` or `Dev` environments, which are a common approach to testing infrastructure related changes in our industry.

## Trade-offs 

### Effort

Cell-based architecture requires a considerable effort to be implemented properly, and it’s not something that an existing system can easily evolve into. In software engineering, we often distinguish between exactly 1 and 1+ problems because the effort required to implement a solution that handles multiples of X can be an order of magnitude bigger than a solution that handles just one of X.  Call-based architecture falls squarely into the 1+ bucket. 

### Hidden shared parts

If you found yourself thinking that there must be something connecting all these independent  parts then you are right. In the real world, nothing can be 100% isolated :). In Octopus Cloud's case, it's the DNS that plays the role of the connective tissue.

## Conclusion

Cell-based architecture is the corner stone of the reliability and scalability of Octopus Cloud and this post tries to provide a high-level overview it. If this post left you with some unanswered questions, then please ask them in the comments section below.

Happy deployments!
