---
title: The architecture of Octopus Cloud
description: Learn how cell-based architecture helped us make Octopus Cloud reliable and scalable.
author: pawel.pabich@octopus.com
visibility: public
published: 2024-09-02-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - Octopus Cloud
  - Engineering
  - Azure
---


Learn how cell-based architecture helped us make Octopus Cloud reliable and scalable.

## What is Octopus Cloud

Octopus Cloud is the easiest way to run Octopus Deploy. It has the same software and functionality as Octopus Server, except we host it for you. You don’t need to download, install, and manage it yourself. Check out [our documentation](https://octopus.com/docs/octopus-cloud) if want to give it a try

ToDo(remove): Please note the advice in our style guide about [adding resources to your post](https://www.octopus.design/932c0f1a9/p/901d2a-blog-content-basics/t/01404d).


## Foundations of Octopus Cloud

Octopus Cloud is a distributed system designed and built on top of Microsoft Azure to withstand partial infrastructure outages. This high-level goal has been translated into the following guiding principles, which the engineering team has been following since the inspection of the product.

### Avoid single point of failure

Octopus Cloud is based on what’s often called a cell-based architecture. A cell represents an independent and isolated part of a system that can be replicated to increase its scalability and reliability. Individual cells can be created, destroyed, upgraded, scaled, or simply broken without affecting the rest of the system.

![Different types of cells in Octopus Cloud](cells.png)

#### Azure region as a cell

Octopus Cloud is currently deployed in [3 different Azure regions](https://octopus.com/docs/octopus-cloud#octopus-cloud-hosting-locations) and can be easily deployed to more regions if we see enough demand there. Resources used by Octopus Cloud in each Azure region are fully isolated from each other. This means that an outage in the Azure West US2 region **will only** affect customers hosted in that region, and **will not** affect customers hosted in West EU and East Australia regions.

#### Reef as a cell

Resources deployed into each Azure region also follow the cell-based architecture. There is more than one reef in each region. A reef is a collection of Azure resources shared by a number of Cloud instances. A Cloud Instance is an Octopus Server instance hosted in Octopus Cloud. Reefs don’t share any resources, so an outage of one of the reefs in a region doesn’t affect all other reefs in that region.  

Fun fact. We picked Reef as the name because this is where octopuses usually live in tropical waters. The other popular contender was Octopus Cloud region, but we felt that it might get easily confused with Azure regions.

#### Cloud instance as a cell

Octopus Server is a monolithic .NET application that requires 3 basic resources to run:
* SQL Server database for structured data (e.g. project data)
* Shared file system for unstructured data (e.g. task logs, packages)
* VM/container for computing

Octopus Cloud provides Cloud instances with an Azure version of each of the required resources:

![Mapping of Octopus Server resource requirements to Azure resources provided by Octopus Cloud](resources.png)

This is where the isolation provided by the strict cell-based approach used at the region and reef level changes from a binary proposition to a wide range of options. 

At one end of the spectrum, we can host a single instance on its own dedicated reef. This is a very powerful option, but we avoid using it because it’s not cost-effective.

At the other end of the spectrum, hundreds of instances can run on a single reef and share a single Azure SQL elastic pool, single AKS cluster, and single Azure storage account. This option works well for Cloud instances that are active at different times of the day and don’t generate a lot of load.

Finally, we have a few options in between those 2 extremes. For example, a busy Cloud instance will have dedicated resources assigned to its database.
 
Each Cloud Instance having its own database and file share allows us to use the built-in Azure tools for managing backup and restore processes so we don’t have to reinvent the wheel in this space.

### Design for gradual rollouts

The cell-based architecture lends itself to gradual rollouts because the infrastructure in each reef and region can be upgraded independently. This reduces the risk of a change causing an outage that affects the whole of Octopus Cloud. As much as nobody likes outages, an outage that affects hundreds of instances is a better place to be than an outage that affects thousands of instances. 

The decision to provision a dedicated file share and a database for each Cloud Instance also helps with the gradual rollout of the Octopus Server changes because each instance can be upgraded and downgraded in isolation. That would not be possible if, for example, more than one Cloud instance shared the same database via database schemas or a similar mechanism that implements multi-tenancy.


### Scalable by default

This is again where cell-based architecture shines. The infrastructure of Octopus Cloud can be easily scaled up by adding new reefs.  Cloud instances can have their resource limits specified on a per-instance basis.

Fun fact. We’ve had an event where Octopus Cloud doubled the number of active Cloud instances within 48-72 hours, and nothing broke. No existing customers were affected. Octopus simply paid a higher hosting bill that month.


### Avoid temporal coupling

Octopus Cloud must be able to perform its primary function, which is keeping Cloud instances up and running, even when some of the non-essential services aren’t available. For example, Cloud instances should continue to operate when:

Octopus Cloud monitoring platform is not available
Octopus Cloud management platform is not available
Octopus Cloud observability platform is not available

In the context of [the CAP theorem](https://en.wikipedia.org/wiki/CAP_theorem),  Octopus Cloud opts for availability over consistency at the infrastructure level, and consistency over availability at the individual Cloud instance level

### Use officially supported APIs only

Octopus Cloud glues together several 3rd party services, which is done by using APIs provided by these services. This might sound like an obvious choice. That said, we’ve extended it also to interactions with internal services and Octopus Server itself.  For example, if Octopus Cloud needs to provide a specific configuration setting to a Cloud instance and that setting already exists in the Octopus Server database but is not exposed via an existing public API, then we will add a new API and will not modify this value directly in the database.

Fun fact: After years of using APIs from major service and Cloud providers, we have learnt that very few of them follow [SemVer](https://semver.org/), and changes to runtime characteristics (e.g., a synchronous operation becomes asynchronous) happen without any notice. 

## Developer productivity

Cell-based architecture can also increase engineering productivity which is often overlooked. Each change can be tested in each own, isolated reef, either locally or in an automated build. There is no need for shared Test or Dev environments, which are common approach to testing infrastructure related changes

## Trade-offs 

### Effort

Cell-based architecture requires considerable effort to implement properly, and it’s not something that an existing system can easily evolve into. In software engineering, we often distinguish between exactly 1 and 1+ problems because the effort required to implement a solution that handles multiples of X can be an order of magnitude bigger than a solution that handles one of X.  Call-based architecture falls squarely into the 1+ bucket. 

### Hidden shared parts

In the real world, nothing can be 100% isolated. In the Octopus Cloud case, DNS plays the role of the connecting tissue.


## Conclusion

Cell-based architecture is the corner stone of the reliability and scalability of Octopus Cloud. 

Has this post left you with some unanswered questions? If so, please ask them in the comments section below.

Happy deployments!
