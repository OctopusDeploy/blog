---
title: Converting an existing application to use rolling deployments
description: This post covers how to convert an existing application to use the rolling deployments pattern in Octopus using Child Steps
author: mark.harrison@octopus.com
visibility: private
published: 2020-07-31
metaImage: rolling-deployments.png
bannerImage: rolling-deployments.png
tags:
 - DevOps
 - Deployment Patterns
 - Product
---

![Rolling Deployments](rolling-deployments.png)

In a previous post this year, I wrote about the [benefits of the rolling deployments pattern](/blog/2020-01/ultimate-guide-to-rolling-deployments/index.md) as a way to reduce application downtime when deploying. Designing an application to fit this deployment pattern is arguably much easier when you’re first creating it. But where do you start with an existing application, and how do you take the application and convert it to use the rolling deployments pattern?

In this post, I'll show you how to convert an existing application to use the rolling deployments pattern in Octopus with the help of [Child Steps](https://octopus.com/docs/deployment-patterns/rolling-deployments#Rollingdeployments-Childsteps).

<h2>In this post</h2>

!toc

## The application

I am going to use [PetClinic](https://github.com/spring-projects/spring-petclinic) as an example and convert this from a process which runs deployment steps sequentially in Octopus, to a rolling deployment process. PetClinic is a sample Spring Boot application written in Java and has 2 main components:

- A web front-end
- A database

:::hint
I don’t explain how to build the PetClinic application in this post. If you are new to building Java applications, we have a number of [Java Guides](https://octopus.com/docs/guides?application=java) which include step-by-step instructions to setup CI/CD pipelines for various tools.
:::

For both the sequential and rolling deployment processes, the PetClinic application and [MySQL](https://www.mysql.com/) database are hosted in [Google Cloud](https://cloud.google.com/gcp). All of the infrastructure including the servers, load balancer and databases are re-created regularly using [Runbooks](https://octopus.com/docs/operations-runbooks)

### Some caveats

It’s important to highlight that this post makes a couple of assumptions about the application set-up:

1. The database is already deployed in a highly available configuration. For more information on MySQL high availability, refer to the [documentation](https://dev.mysql.com/doc/mysql-ha-scalability/en/ha-overview.html).
1. Load balancer sticky sessions

## Sequential deployment process

## Converting to a rolling deployment process


:::success
**Sample Octopus Projects**
You can see the before and after of the project conversion discussed here in our samples instance.

- [PetClinic project - with no rolling deployments](https://g.octopushq.com/PatternRollingSamplePetClinicNoRollingDeploy)
- [PetClinic project - with rolling deployments](https://g.octopushq.com/PatternRollingSamplePetClinicRollingDeploy)
- [PetClinic Infrastructure Runbooks](https://g.octopushq.com/PatternRollingSamplePetClinicIacRunbooks)
:::

## Conclusion

TODO

## Learn more
- [Octopus Rolling deployments docs](https://octopus.com/docs/deployment-patterns/rolling-deployments)
- [Octopus CI/CD guides](https://octopus.com/docs/guides)
