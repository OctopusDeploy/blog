---
title: Solutions Engineering - When being a jack-of-all-trades is a good thing
description: Solutions Engineering - When being a jack-of-all-trades is a good thing.
author: shawn.sesna@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - tag
---

In the tech world, being a "jack of all trades master of none" often carries with it a negative conotation.  It means that you have a broad wealth of knowledge, but you aren't knowledgable in any one area which is seen as more of a hinderence than a boon.  However, the phrase "jack of all trades master of none" is incomplete, the full saying is this, "Jack of all trades master of none, though oftentimes better than master of one."  The full quote is very appropriate when it comes to the role of the Solutions Engineer (SE), especially at Octopus Deploy.

## The Solutions Engineering role
At Octopus Deploy, it is important for the SEs to know the entire Software Devevelopment Life Cycle (SDLC) from soure control all the way to the deployed application and what hosts it.  The knowledge that we need falls into the following categories:

- Source Control Management
- Build Server
- Artifact Repository
- Application Host


### Source Control Management
In terms of Source Control Management (SCM), it is generally agreed that `git` is the predominate technology.  SCM server technology such as Azure DevOps, BitBucket, GitLab, GitHub, etc... implement git as either an option or the only method of source control.  While it's not often that an Octopus SE needs to work with a customer in an SCM, the [Build Information](https://octopus.com/docs/packaging-applications/build-servers/build-information) feature utilizes commit information so having knowledge in how git works is a must.

Source control is the beginning of the SDLC and feeds directly into the next topic, `Build Servers`.

### Build servers
Build servers compile software on independent runners to ensure the code builds correctly and there aren't any missing dependencies.  Similar to SCM, there are many choices when it comes to build servers.  Some are part of all-in-one platforms such as Azure DevOps or GitLab while others are dedicated build servers such as CircleCI or TeamCity.  Build servers are often the first integration point of a customer with Octopus Deploy so it is important for the SEs to have knowledge in many build servers and how to construct their respective build definitions.  In instances where Octopus does not have an available plugin, SEs need to know how to invoke the Octopus CLI or make API calls from the build server.

Build servers often produce artifacts that need to be deployed to the application host.  Whether the artifact is a container or an executable, the artifacts need to be stored in an Artifact Repository.

### Artifact Repository
While Octopus Deploy comes with a built-in repository (except for containers), many customers opt to use other repository technology for various reasons.  The all-in-one platforms often contain their own repositories, however, there are other, purpose-built repositories such as Artifactory or Nexus.  While the repository types implement the same protocols (Nuget, Maven, Docker registry, etc...), each platform implements connection to their endpoints differently.  This requires Octpous SEs to be familiar with many different repository technologies to assist customers in connecting Octopus.

### Application Host
The last part of the SDLC is the deployment of an application to an application host.  This category is perhpas the most extensive in what an Octopus SE needs to be familiar with.  Some applicaiton host technology can have multiple solutions; the same Java application can be hosted on Tomcat, Wildfly, IBM Websphere Liberty, or Payara (and perhaps others).  However, each technology deploys the application to the server differently.  In the container world, cloud providers have their own implementation of container orchestration such as AWS Elastic Container Service (ECS) or Azure Container Instances (ACI) as well as their implementation of Kubernetes (Azure Kubernetes Service (AKS), AWS Elastic Kubernetes Service (EKS), and Google Kubernetes Engine (GKE)).  In addition, there are some cloud providers who offer managementless Kubernetes like Azure Container Apps (magementless in this context means that Azure manages the cluster you deploy to, all you have to worry about is deploying the containers).  Other things we need to know about are methods for automating database deployments or even more specialized categories such as automation of the deployment of SQL Server Reporting Services (SSRS) reports or SQL Server Integration Services (SSIS) packages.  Database deployments carries with it the need to know about the database technology being deployed to; Oracle is very different from SQL Server.

## Conclusion
Needing to know a little about a lot is not unique to Octopus or a Solutions Engineering role.  Having broad knowledge versus depth can be a asset, especially when it comes to needing to help someone integrate technologies together.  To those who identify as the jack of all trades, know that your skills are indeed useful and needed.

Happy Deployments!


