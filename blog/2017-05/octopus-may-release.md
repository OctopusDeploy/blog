---
title: "Octopus May Release 3.13"
description: TODO  
author: mark.siedle@octopus.com
visibility: private
tags:
 - New Release 
 - Azure Service Fabric
---


## Introducing Azure Service Fabric support

We are excited to announce that Octopus now includes first-class support for [Deploying Azure Service Fabric applications](https://octopus.com/docs/deploying-applications/deploying-to-service-fabric).

Since the [RFC](https://octopus.com/blog/rfc-azure-service-fabric) earlier this year, we've been busy creating new step templates to help you connect to and deploy your Service Fabric cluster applications.

These new Service Fabric steps can now assist you with:
- Deploying a Service Fabric App ([learn more](https://octopus.com/docs/deploying-applications/deploying-to-service-fabric/deploying-a-package-to-a-service-fabric-cluster))
- Running a Service Fabric SDK PowerShell Script ([learn more](https://octopus.com/docs/deploying-applications/custom-scripts/service-fabric-powershell-scripts))

Both steps require connection to a cluster. As such, we've included support for security modes including unsecure, Client Certificates and Azure Active Directory.

:::hint
**Service Fabric SDK**
Due to Service Fabric dependencies, you will need to manually install the [Service Fabric SDK](https://g.octopushq.com/ServiceFabricSdkDownload) onto your Octopus Server. Then you can then start using Octopus to help orchestrate your Service Fabric application deployments.
:::

### Want to learn more?

You can learn more about these new features from our main [Deploying to Service Fabric](https://octopus.com/docs/deploying-applications/deploying-to-service-fabric) documentation. 

We also have a new guide explaining [Continuous Integration for Service Fabric](https://octopus.com/docs/guides/service-fabric) where you can learn how Octopus Deploy fits into a Continuous Deployment pipeline for you Service Fabric applications.

### Thanks to you!

We'd like to say a big thanks to the community for all their feedback around this feature. We're interested to hear your thoughts.