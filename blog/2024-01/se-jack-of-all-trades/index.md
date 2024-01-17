---
title: "Solutions engineer - the jack of all trades"
description: Learn why being a jack of all trades is great when you're a solutions engineer.
author: shawn.sesna@octopus.com
visibility: private
published: 2024-01-22-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - Engineering
  - DevOps
---

In the tech world, being a "jack of all trades" is often taken negatively as people finish the sentence with "master of none".  It means you have a broad but shallow knowledge of many things, but your knowledge doesn't go deep in any one area. People consider this a hindrance rather than a boon.

However, the phrase "jack of all trades, master of none" is incomplete. The full saying goes, "Jack of all trades master of none, though oftentimes better than master of one."  This is appropriate when it comes to the role of solutions engineer (SE), especially at Octopus Deploy.

## The solutions engineering (SE) role

At Octopus Deploy, the SEs need to know the entire software development lifecycle (SDLC), from source control to the deployed application and what hosts it. The knowledge we need falls into the following categories:

- Source control management
- Build server
- Artifact repository
- Application host


### Source control management

Git is the most common technology for source control management (SCM). SCM server technology, like Azure DevOps, BitBucket, GitLab, and GitHub, implements Git as an option or the only method of source control.  It's rare that an Octopus SE needs to work with a customer in an SCM. However, our [build information](https://octopus.com/docs/packaging-applications/build-servers/build-information) feature uses commit information, so it's vital to know how Git works.

Source control is the beginning of the SDLC and feeds directly into the next topic, build servers.

### Build servers

Build servers compile software on independent runners. They  ensure the code builds correctly and there aren't any missing dependencies. Like SCM, you have many build server choices. Some are part of all-in-one platforms, like Azure DevOps or GitLab. There are also dedicated build servers, like CircleCI or TeamCity. 

Build servers are often the first connection point with Octopus Deploy, so our SEs need to know many build servers and how to construct their build definitions. In the rare case Octopus doesn't have a plugin, SEs need to know how to invoke the Octopus CLI or make API calls from the build server.

Build servers often produce artifacts that need to deploy to the application host. Whether the artifact is a container or an executable, an artifact repository needs to store the artifacts.

### Artifact repository

While Octopus comes with a built-in repository, some customers use other repository tools. All-in-one platforms often have their own repositories. There are also other, purpose-built repositories, like Artifactory and Nexus. While the repository types implement the same protocols (like Nuget, Maven, and Docker registry), each platform connects to its endpoints differently. Octopus SEs need to be familiar with many different repository technologies to help customers connect with Octopus.

### Application host

The last part of the SDLC is the deployment of an application to an application host. This category is the most extensive we need to be familiar with. 

Some application host technologies have many solutions. You can host the same Java application on Tomcat, Wildfly, IBM Websphere Liberty, or Payara (and perhaps others). However, each technology deploys the application to the server differently. 

In the container world, cloud providers have their own implementation of container orchestration. These include AWS Elastic Container Service (ECS) or Azure Container Instances (ACI), and their implementation of Kubernetes. This includes Azure Kubernetes Service (AKS), AWS Elastic Kubernetes Service (EKS), and Google Kubernetes Engine (GKE). 

Some cloud providers also offer managementless Kubernetes, like Azure Container Apps.(Managementless here means that Azure manages the cluster you deploy to. All you have to worry about is deploying the containers.) 

Other things we need to know include:

- Methods for automating database deployments
- Specialized categories like automating deployments of SQL Server Reporting Services (SSRS) reports or SQL Server Integration Services (SSIS) packages

For Database deployments, we need to know about the database technology you're deploying to. For example, Oracle is very different to SQL Server.

## Conclusion

Needing to know a little about a lot is not unique to Octopus or a solutions engineering role. Having broad rather than deep knowledge is an asset. This is especially the case when helping someone integrate technologies together. If you identify as a jack of all trades, know that your skills are indeed useful and needed.

Happy deployments!