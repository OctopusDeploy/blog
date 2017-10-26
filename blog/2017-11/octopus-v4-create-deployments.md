---
title: Octopus Deploy 4.0 - Create Deployment Page 
description: The Create Deployment page has been remodelled as part of Octopus version 4.0  
author: michael.richardson@octopus.com
visibility: private
tags:
 - New Releases
---

While working on Octopus v4, many pages were essentially a direct port to the new look-and-feel. But there were a few where we took the opportunity to re-think the design. 

One of the latter was the page to create a deployment.

The existing page was originally created to deploy a release to a single environment.  It was later extended to allow deploying to multiple environments. Then the multi-tenancy feature came along, and we bolted on the ability to create deployments for one or more tenants. 

**TODO: INSERT IMAGE**

If we're honest, it was overdue for a re-design.

**TODO: INSERT IMAGE**

## Introducing the new create deployment page

The most significant changes are seen when deploying a release to multiple environments or tenants. 

We conceptually split the page into two sections: the North and the South.

In the image below, the _Preview and Customize_ separator forms the Equator.

### North

In the North, you select the environments and tenants (if it is a multi-tenant project) you wish to deploy to.

### South

The South section hopefully makes it clear this will result in _multiple_ deployments being created. This was something the existing design did not communicate effectively.

Expanding a deployment allows you to preview which deployment steps will be executed, and on which targets. You can also include\exclude specific deployment targets here.  
