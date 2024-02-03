---
title: Deploying database updates with Octopus Deploy and the Liquibase Execution Container
description: Introducing the Liquibase Execution Container.
author: shawn.sesna@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: to-be-added-by-marketing
bannerImage: to-be-added-by-marketing
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - DevOps
  - Company
  - Product
  - Engineering
---

Up until now, deploying database updates using the [Liquibase](https://liquibase.com) product required either the Liquibase CLI be installed on the machine performing the updates or by ticking the `Download Liquibase?` box on the [Liquibase - Run Command](https://library.octopus.com/step-templates/36df3e84-8501-4f2a-85cc-bd9eb22030d1/actiontemplate-liquibase-run-command) Community step template to dynamically download Liquibase and any dependencies at deploy time.  This post introduces a publicly available container image that can be used with the [Execution Containers](https://octopus.com/docs/projects/steps/execution-containers-for-workers) feature to execute Liquibase.

## Using the Liquibase container image

The [octopuslabs/liquibase-workertools](https://hub.docker.com/r/octopuslabs/liquibase-workertools) container image comes with most of the components necessary to perform a deployment including:
- Liquibase
- Java
- PowerShell
- AWS CLI (to support AWS IAM authentication)

When selecting a database type that the Liquibase product does not ship with, such as Cassandra or MongoDB, the `Liquibase - Run Command` template will automatically detect if it is being run within a container and download any missing dependencies.


## Updating your process

If you're currently using the `Liquibase - Run Command`, the only change you need to make is updating the `Container Image` section from **Runs directly on a worker** to **Runs inside a container, on a worker** and specify the image as `octopuslabs\liquibase-workertools` (if you don't have an [External Feed](https://octopus.com/docs/packaging-applications/package-repositories/docker-registries) for Docker Hub, you will need that as well).

![Select execution container](octopus-liquibase-container.png)

## Conclusion

We strive to make using Octopus Deploy as easy as possible.  Development of the `octopuslabs\liquibase-workertools` image makes integrating the Liquibase product into your deployment process all that much easier, especially when coupled with Octopus Cloud [Dynamic Workers](https://octopus.com/docs/infrastructure/workers/dynamic-worker-pools).

Happy deployments!
