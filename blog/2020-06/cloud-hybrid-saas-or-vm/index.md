---
title: Octopus Deploy in the Cloud, Virtual Machine or SaaS
description: Determine the right Octopus Deploy installation to use in the cloud
author: lianne.crocker@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
 - Product
 - DevOps
---

Once you've decided to move to cloud infrastructure, a common query at Octopus is whether to move to Octopus Server on a virtual machine or to use Octopus Cloud.  

We're going to take a look at the reasons why you might choose one or the other.  Key considerations will most likely be cost and security, but alongside this, attention should also be given to how you'll use Octopus.  The information in this post will help you in your deliberations. 

:::hint

**Octopus Cloud** refers to the [SaaS](https://en.wikipedia.org/wiki/Software_as_a_service) offering from Octopus Deploy and **Octopus Server** to the self-hosted version of Octopus Deploy.  In the context of this post, **Octopus Server** resides on a virtual machine in the cloud.    

- [Octopus Cloud](http://g.octopushq.com/OctopusCloudPricing)
- [Octopus Server](http://g.octopushq.com/ProductDownloadPage)

:::

!toc

## Architecture

****  A section giving a brief overview of the architecture?  ***


## Management

Opting for Octopus Cloud reduces the number of administration tasks you have to carry out on your Octopus instance; however, there are some reasons you might want the greater control that Octopus Server offers. 

### Retention policies and storage

Octopus [retention policies](http://g.octopushq.com/RetentionPolicies) have a default configuration on all Octopus instances; we recommend reviewing them to suit your requirements. 

When configuring Octopus Server on a VM, you select the storage that Octopus uses for the [server folders](http://g.octopushq.com/OctopusServerFolders); this means you can set the storage limits that suit you for these folders and configure retention policies accordingly.

Octopus Cloud provides plenty of storage for most businesses; however, the amount of storage you can use has a [limit](ttp://g.octopushq.com/AcceptableUsage).  We've added monitoring to the Technical section of the Octopus instance management page to view the details and login to your Octopus Account. Then, on your instance, select {{ Manage,Resource Usage }}, this is updated every 24 hours. 

![cloud-limits](cloud-limits.png "width=500")

Octopus has a built-in package repository available to all instances, in addition to this external repository feeds can be configured to serve your [deployment packages](http://g.octopushq.com/OnboardingPackageRepositoriesLearnMore).  You might choose to use an external feed to aid in [multi-region deployments](#regional-package-feeds) or if your package feed storage requirements are considerable.

### Maintenance window

One of the great things about Octopus Cloud is that you don't have to worry about how and when to perform system maintenance and [upgrades](http://g.octopushq.com/OctopusUpgradeDoc), Octopus manage those for you and, additionally, you're able to choose a maintenance window to suit your business and region.

Outage window information is in the Technical section of the Octopus instance management page:

![cloud-outage-window](cloud-outage-window.png "width=500")

Octopus Server allows you to be even more specific about when you perform your system maintenance and upgrades.  If you implement [high availability](http://g.octopushq.com/HighAvailability) then you can have zero downtime while maintenance is carried out.  Some customers choose to automate maintenance and upgrade tasks by using [runbooks](http://g.octopushq.com/OnboardingRunbooksLearnMore) in [another instance of Octopus](http://g.octopushq.com/UpgradeOctopusWithOctopus), a free Octopus Cloud instance works perfectly here. 

### Managing infrastructure

Choosing Octopus Server allows you to use infrastructure that fits into your business's operational architecture; any infrastructure management already in place can also be leveraged for your Octopus instance.  You can choose when or if to scale out/up.  It is also your responsibility to ensure that the [infrastructure](http://g.octopushq.com/ManagingInfrastructure) performs well, is maintained and recoverable.

Using Octopus Cloud allows you to take a step back from infrastructure management; it's our responsibility to ensure that the infrastructure is robust and scaling according to your usage. 

Both Octopus Server and Octopus Cloud allow you to add your own [worker](http://g.octopushq.com/OnboardingWorkersLearnMore) machines in addition to the [dynamic worker](http://g.octopushq.com/DynamicWorkerPools) (on Octopus Cloud) and [built-in worker](http://g.octopushq.com/BuiltinWorker) (on Octopus Server).  When using Octopus Server, we recommend using workers where possible to reduce the resource usage on the Octopus server and, as Octopus executes under a privileged account, if we can offload work to another machine, it's wise to do so for added security.

### Backups

Backups are included as part of the managed service with Octopus Cloud.  The database, built-in package repository, and configuration are backed up.  If you use an external package feed, then it is your responsibility to ensure backups are configured.

When managing your infrastructure as part of an Octopus Server installation, you can design your [backup](http://g.octopushq.com/BackupRestore) and recovery plan to match your existing infrastructure plans. 

### Monitoring

Octopus [subscriptions](http://g.octopushq.com/Subscriptions) can be used in both Octopus Cloud and Octopus Server to notify of events to watch for changes in Octopus - a change to a specific project process, a user amendment, the addition of a deployment target, the list is long!

The [REST API](#octopus-rest-api) has a [reporting](http://g.octopushq.com/Reporting) endpoint that you can use to create a dashboard, available in both Octopus Cloud and Octopus Server. 

Octopus Server also offers the ability to [send logs to a central log tool](https://help.octopus.com/t/how-can-i-configure-octopus-deploy-to-write-logs-to-a-centralized-logger-such-as-seq-splunk-or-papertrail/24551) by adding a log target to the nlog config file. 

## Octopus REST API

Octopus Deploy is built API-first, anything you can do through the user interface can be done using the [API](http://g.octopushq.com/RestAPI), this is true for both Octopus Cloud and Octopus Server.  

We have an array of samples accessing the API in our Github [OctopusDeploy/Api](http://g.octopushq.com/OctopusApiSamples) repository.

## Database

Octopus uses a SQL Server database.  The Octopus Cloud database is not accessible to customers, so all interactions with Octopus must be done using the web interface or via the API.  The great thing about this is that the database maintenance, security, and patching is all done by us!

Octopus Server on a cloud VM gives you several options for database hosting.

- **VM** - Host SQL Server on a virtual machine.
- **PaaS** - [Google Cloud SQL](https://cloud.google.com/sql/docs/sqlserver), [Azure SQL](https://azure.microsoft.com/en-gb/services/sql-database/) or [AWS RDS for SQL Server](https://aws.amazon.com/rds/sqlserver/) for example. 
- **Docker** - Use SQL Server within a [Docker](https://www.docker.com/) container, this could be on a Windows or Linux virtual machine or a cloud container hosting platform.   

The benefit of managing your database offers more the choice of more lenient [retention policies](http://g.octopushq.com/RetentionPolicies), the ability to use a clustered database, and managing your backups.

## Security

With Octopus Cloud, we secure the infrastructure and carry out penetration testing to ensure the security and integrity of the  Octopus instances.  In this case, you are responsible for:

- How you connect Octopus to your infrastructure.
- How you identify your users and control their activities within Octopus.
- How you handle sensitive information within Octopus.

When using Octopus Server, it is your responsibility to [harden](https://octopus.com/docs/security/hardening-octopus) the Octopus instance infrastructure.  This level of control may be beneficial if there are industry-specific regulations to which you are required to adhere.  In this case, you also taking responsibility for:

- How you harden the underlying server operating system.
- How you protect the Octopus Server files on the operating system.
- How you store files generated by Octopus Server.
- How you secure your SQL Database and protect the data generated by Octopus Server.
- How you expose your Octopus Server to your infrastructure.
- How you identify your users and control their activities within Octopus.
- How you handle sensitive information within Octopus.

When adding deployment targets and worker machines to Octopus Cloud or Octopus Server infrastructure, their security and integrity is your responsibility.

### Authentication Providers

Octopus Cloud [authentication](http://g.octopushq.com/AuthenticationProviders) uses [Octopus ID](http://g.octopushq.com/AuthOctopusID), allowing you to use the same account that you use to sign in at [octopus.com](https://octopus.com).  Octopus ID supports username and password or can use a Google or Microsoft account.

![octopus-id](octopus-id.png "width=500")

Octopus Server has several authentication providers available to use:

- username and password
- [Active Directory](http://g.octopushq.com/AuthAD)
- [Azure Active Directory](hhttp://g.octopushq.com/AuthAzureAD)
- [Google account](http://g.octopushq.com/AuthGoogleApps)
- [Okta](http://g.octopushq.com/AuthOkta)

If desired, multiple providers can be configured concurrently. 

### IP restrictions

Using Octopus Server on a VM allows you to configure the IP and DNS to what suits your business' requirements.  You may choose to configure a load balancer to handle traffic to the Octopus web interface; this is especially useful if configuring [HA](#high-availability).

An Octopus Cloud instance has a range of [static IP addresses](http://g.octopushq.com/CloudStaticIP) shared among customers in the same Azure region, and these can found in the Technical section of the Octopus instance management page:

![cloud-ips](cloud-ips.png "width=500")

## Networking

Connecting [Polling Tentacles](http://g.octopushq.com/PollingTentacle) over WebSockets to Octopus Cloud is not currently possible, however, we do have plans to support Polling Tentacles connecting on port 443 soon.

### Proxy support

Both Octopus Server and Octopus Cloud have support for [proxies](http://g.octopushq.com/ProxySupport).  You can specify a proxy server for Octopus to use when communicating with a Tentacle or SSH Target. You can also specify a proxy server when a Tentacle and the Octopus server make web requests to other servers.

## Region

Using Octopus Server on a VM lets you host your Octopus instance in your choice of geographic region.  

Octopus Cloud also offers you the option of hosting your instance in a region that suits you, we use Microsoft Azure to host Octopus Cloud. At the time of writing, we offer the following regions:

- West US 2
- West Europe
- Australia East

The region is selected when your first create your Octopus Cloud instance.

![cloud-region](cloud-region.png "width=500")

Should we introduce a new region that would suit your needs better, please get in touch so that we can move your instance to that region. 

### Regional Package feeds

If you operate in multiple regions and are concerned about network latency issues while transferring large packages during deployments, we'd recommend opting for an external [package feed](http://g.octopushq.com/OnboardingPackageRepositoriesLearnMore) and either replicating it across multiple regions and using DNS aliasing to have the tentacle fetch packages from geographically local package repositories.  Or, an even simpler solution than that is to have two separate but identical package repositories and have the [Feed ID as a variable](http://g.octopushq.com/DynamicPackageFeeds).  Package repositories work in this way for both Octopus Server and Octopus Cloud. 

## High Availability

[High Availability](http://g.octopushq.com/HighAvailability) (HA) can be configured for Octopus Server to enable you to run multiple Octopus Servers, distributing load and tasks between them.  

While Octopus HA is not available for Octopus Cloud, all instances are provisioned as a [Linux container running on AKS](https://octopus.com/blog/octopus-cloud-v2-why-kubernetes) (Azure's managed Kubernetes).  [Kubernetes](https://kubernetes.io/) helps us achieve the resilience and scaling flexibility to make Octopus Cloud robust and performant.

## Other considerations

### Tentacles

An Octopus tentacle machine can be configured to be either [polling](http://g.octopushq.com/PollingTentacle) or [listening](http://g.octopushq.com/ListeningTentacle).  The type of tentacles you select for your deployments will depend on your business requirements. You should also consider if the configuration of these machines has any impact on the infrastructure chosen for your Octopus instance. 

### Octopus in a container

An alternative to running Octopus Server on a virtual machine is to run it in a container.  In fact, this is how we manage our Octopus Cloud instances.  The [Windows container](http://g.octopushq.com/DockerWindows) for Octopus is fully supported, and the [Linux container](http://g.octopushq.com/DockerLinux) is, at the time of writing, part of our Early Access Program. 

## Roundup

This post has hopefully provided clarity to your decision-making process when moving to cloud infrastructure.  Whatever your business requirements, Octopus Deploy makes release management easy and simplifies even the most complex deployments wherever you deploy your software. Runbook automation minimizes outages and gives you control over your infrastructure and applications.  

If you have any questions or would like advice on how Octopus can work for you please drop us a line at [advice@octopus.com](mailto:advice@octopus.com)

## More information

- [An explanation of worker security and scoped worker pools](http://g.octopushq.com/WorkerSecurityScopedWorkerPools)
- [Execution containers for workers](http://g.octopushq.com/ExecutionContainersForWorkers)
- [Moving octopus](http://g.octopushq.com/MovingOctopusComponents)