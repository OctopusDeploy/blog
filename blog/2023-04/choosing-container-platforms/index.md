---
title: Choosing a container platform
description: A look at different container platforms and their suitability.
author: nikita.dergilev@octopus.com
visibility: public
published: 2023-04-19-1400
metaImage: pblogimage-top8containerregistries-2022.png
bannerImage: blogimage-top8containerregistries-2022.png
bannerImageAlt: Three-tiered shelf housing eight blue containers.
isFeatured: false
tags: 
  - DevOps
  - Containers
  - Kubernetes
  - Docker
---

There are many container platforms to choose from. Finding the right platform for your project boils down to:

- Where you want or need to run them
- Your team's knowledge at the time
- Flexibility needed
- Company policy
- Budget

It's important to check the pros and cons of each platform and weigh them against your product's needs.

In this blog, we help break down the suitability of each platform.

## Containers on-premise

There are 2 main strategies for running containers on-premise, distributed computing and using private data centers.

### Distributed computing

Distributed computing is when you deploy to many similar (but not always indentical) targets to counter network risks or high remote running costs. It's common for businesses like fast-food restaurants, gyms, and hospitals.  

The main challenges with distributed computing are:

- Deployment standardization - You must manage differences between sites, like network speeds, maintenance windows, or customer's policies
- Maintenance costs - Though hardware might be cheap or useful for cross-purposes, don't forget cost of maintenance or service outages
- Support and recovery options - Consider who's supporting the local hardware or running recovery procedures

Let's look at the best container options for distributed computing.

#### K3s

We recommend [Kubernetes](https://kubernetes.io/) as it has good community support and offers great standardization.

[K3s](https://k3s.io/) is a lightweight and easy-to-install Kubernetes distribution. It's suitable for small to medium-sized companies that don't need advanced Kubernetes features. K3s still have standard Kubernetes features like scalability and network management, but with less complexity.

You also still define resources using the same YAML files you would for any other Kubernetes setup, so it's easy to find examples and community help.

#### Docker

If you don't need to deploy many containers, you can use [Docker](https://www.docker.com/) on a local computer to run your applications.

#### Docker Swarm

[Docker Swarm](https://docs.docker.com/engine/swarm/) is Docker's container orchestration tool.

Swarm is useful if you're already using Docker's other services as it's easy to use, but there are better long-term solutions.

#### Nomad

[Nomad](https://www.nomadproject.io/) is a container orchestration platform by HashiCorp.

Nomad is less popular than Kubernetes but has all the features you need for running containers on-premises.

### Private data centers

Private data centers are the most sophisticated and centralized hosting option. Despite being less popular due to cloud services, they’re ideal for specialist security, resource, and location needs.

The main challenges with private data centers are:

- Expense - Running a private data center can cost you not only financially, but also in maintenance and the expertise needed
- Disaster recovery - No data center offers 100% availability, so you might need to consider extra data centers for disaster cutover

Let's look at the best container options for private data centers.

#### Kubernetes

[Kubernetes](https://kubernetes.io/) is a highly scalable and flexible container orchestration tool. It's ideal for large and complex applications.

Kubernetes is suitable for mid-to-large companies with DevOps teams who can manage its complexity.

### Docker Swarm or Nomad

Both [Docker Swarm](https://docs.docker.com/engine/swarm/) and [Nomad](https://www.nomadproject.io/) are solid options for private data centers as they're simpler to manage than Kubernetes.

Either is a suitable alternative if you lack time to find or train a team on Kubernetes.

## Containers in the cloud

In most cases, running containers in a cloud solution is the best option, especially if you don't have specific needs.

### Cloud virtual machines (VMs)

You could use a cloud VM with any cloud provider to host and run a container platform. In theory, this approach is like managing a private data center without physical hardware.

However, using a cloud VM means you're responsible for setup and maintenance, especially when your app needs to scale. We only recommend this solution if you have unique infrastructure or security needs. On the other hand, it may make recovery processes easier to manage.

### Platform as a Service (PaaS) options

Most of the most popular cloud services offer PaaS options for containers. PaaS services offer power and flexibility but they can be expensive too.

PaaS containers work best for mid to large-size engineering teams with people dedicated to maintenance.

Let's look at some PaaS options.

#### Kubernetes

[Kubernetes](https://kubernetes.io/) again! It's the most popular cloud option, given most cloud providers support it or have their own version. For example, you have:

- [Azure Kubernetes Service (AKS)](https://azure.microsoft.com/en-au/products/kubernetes-service/)in Microsoft Azure
- [Elastic Kubernetes Service (EKS)](https://aws.amazon.com/eks/) in Amazon Web Services (AWS)
- [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine) in Google Cloud Services (GCS)
- [IBM Cloud Kubernetes Services](https://www.ibm.com/cloud/free/kubernetes?utm_content=SRCWW&p1=Search&p4=43700074964128080&p5=e&gclid=CjwKCAjwitShBhA6EiwAq3RqA9qVQzKIVn3EKo3rY-nTUogVF5ajHpZEU_NzGNFy-dy8dIy1meYiaBoCNmgQAvD_BwE&gclsrc=aw.ds) in IBM Cloud Foundry

These services make Kubernetes easier to use, especially thanks to:

- Community support
- Example processes
- Popular applications already deployable to containers through methods like Helm charts

You'll still need to consider:

- Vendors may support Kubernetes differently - Make sure your cloud service offers the features and updates you need
- Supporting tools - Cloud providers may have options like container registries and load balancing tools
- Understanding of Kubernetes - A cloud provider will reduce Kubernete's complexity, but you'll still need to understand it

#### Vendor alternatives

Some cloud providers have or support alternative platforms, like [Azure Red Hat OpenShift](https://azure.microsoft.com/en-us/products/openshift/).

OpenShift is a container platform built on top of Kubernetes. It offers features like:

- Automated deployments
- Application scaling
- Security management

### Proprietary cloud platforms

Finally, you can use a proprietary platform developed by a cloud provider.

Example services include:

- [Elastic Container Service (ECS)](https://aws.amazon.com/ecs/) in Amazon Web Services (AWS)
- [Azure Container Apps](https://azure.microsoft.com/en-au/products/container-apps/) in Microsoft Azure

The main challenges with private data centers are:

- Whether the provider supports the solution well enough - Check when the vendor introduced the service, if it's still supported, the userbase size, and if there's a development roadmap
- Hidden limits - Some services may limit how many containers you can deploy, so make sure the service allows everything your software needs
- Tool compatability - Make sure the service works alongside your CI/CD tooling to avoid custom scripting

Though these options tend to lack the flexibility of other platforms, they're easier to use. That makes them a good choice for small to medium-sized engineering teams or simple applications. Of all the options, we recommend considering proprietary services if they suit your software. Otherwise, if limitations are a problem, Kubernetes should be your next step.

## Hybrid setups

Organizations may need to use more than one container platform. For example, you might need to use different cloud services at the same time or use cloud and private data centers together.

In these situations, you should standardize as much as possible. In this case, look for the platforms best supported by your provider, though it's likely you'll land on Kubernetes as it's the best for standardization.

## Octopus can help with container deployments

Regardless of the container platform you opt for, Octopus can help in the following ways:

- Built-in multi-tenancy helps with distributed computing so you can deploy to any combo of customers with ease.
- Standardize deployments for different customers and infrastructure. Use snapshots and variables to manage environmental differences.
- Though you can use YAML, Helm Charts, or customer scripts, anyone can create container deployments in Octopus's easy-to-use UI.

[Read more about how Octopus can help with even the most complex deployments](https://octopus.com/).

## Conclusion

In this post, we covered the different container platforms and highlighted scenarios where they would be a good fit.

For more on containers:

- See everything you need to [get started with containers](https://octopus.com/blog/get-started-containers)
- Follow our guide for [building and deploying a Java app with Docker, Google, Azure, and Octopus](https://octopus.com/blog/deploying-java-app-docker-google-azure)
- Read about [the container registries we recommend](https://octopus.com/blog/top-8-container-registries)