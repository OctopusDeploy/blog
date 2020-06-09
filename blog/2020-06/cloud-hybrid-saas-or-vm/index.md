---
title: Comparing self-hosted Octopus in a Cloud VM vs Octopus Cloud
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

When considering moving Octopus to cloud infrastructure, a common question we receive is whether to move to Octopus Server on a virtual machine or to use Octopus Cloud.

We're going to take a look at the reasons why you might choose one or the other.  Key considerations will most likely be cost and security, but attention should also be given to how you'll use Octopus.  The information in this post will help you in your deliberations. 

:::hint

**Octopus Cloud** refers to the Octopus Cloud [SaaS](https://en.wikipedia.org/wiki/Software_as_a_service) offering from Octopus Deploy and **Octopus Server** to the self-hosted version of Octopus Deploy.  In the context of this post, **Octopus Server** resides on a virtual machine in the cloud.

- [Octopus Cloud](http://g.octopushq.com/OctopusCloudPricing)
- [Octopus Server](http://g.octopushq.com/ProductDownloadPage)

:::

!toc

## Architecture

Before I get into the details, let's look at the architecture for the two incarnations of Octopus we're discussing. 

**Server**

Octopus Server runs as a Windows Service; this provides access to the [Octopus REST API](https://octopus.com/docs/octopus-rest-api) and [Octopus Web Portal](https://octopus.com/docs/getting-started#the-octopus-web-portal).  It connects to a SQL Server database and uses [file shares](http://g.octopushq.com/MovingOctopusComponents) to store task logs, artifacts, and packages.

**Cloud**

Octopus Cloud uses a Linux container running on [AKS](https://octopus.com/blog/octopus-cloud-v2-why-kubernetes) with the database hosted in Azure SQL.  The file shares use Azure Cloud Storage.

## Management

Opting for Octopus Cloud reduces the number of administration tasks you have to carry out on your Octopus instance; however, there are some reasons you might want the greater control that Octopus Server offers. 

### Retention policies and storage

Octopus [retention policies](http://g.octopushq.com/RetentionPolicies) have a default configuration on all Octopus instances; we recommend reviewing them to suit your requirements. 

**Server**

When configuring Octopus Server on a VM, you select the storage that Octopus uses for the [server folders](http://g.octopushq.com/OctopusServerFolders); this means you can set the storage limits that suit you for these folders and configure retention policies accordingly.  

**Cloud**

Octopus Cloud provides plenty of storage for most businesses; however, the amount of storage you can use has a [limit](ttp://g.octopushq.com/AcceptableUsage).  Where long term retention is required for regulatory compliance, you may find this a reason to select Octopus Server over Octopus Cloud.  We've added monitoring to the Technical section of the Octopus instance management page to view the details and login to your Octopus Account. Then, on your instance, select {{ Manage,Resource Usage }}, this is updated every 24 hours. 

![cloud-limits](cloud-limits.png "width=500")

:::hint

Octopus has a built-in package repository available to all instances, in addition to this external repository feeds can be configured to serve your [deployment packages](http://g.octopushq.com/OnboardingPackageRepositoriesLearnMore).  You might choose to use an external feed to aid in [multi-region deployments](#regional-package-feeds) or if your package feed storage requirements are considerable.

:::

### Maintenance window

**Cloud**

One of the great things about Octopus Cloud is that you don't have to worry about how and when to perform system maintenance and [upgrades](http://g.octopushq.com/OctopusUpgradeDoc), Octopus manage those for you and, additionally, you're able to choose a maintenance window to suit your business and region.

Outage window information is in the Technical section of the Octopus instance management page:

![cloud-outage-window](cloud-outage-window.png "width=500")


**Server**

Octopus Server allows you to be even more specific about when you perform your system maintenance and upgrades.  If you implement [high availability](http://g.octopushq.com/HighAvailability) then you can have zero downtime while maintenance is carried out.  Some customers choose to automate maintenance and upgrade tasks by using [runbooks](http://g.octopushq.com/OnboardingRunbooksLearnMore) in [another instance of Octopus](http://g.octopushq.com/UpgradeOctopusWithOctopus), a free Octopus Cloud instance works perfectly here. 

### Managing infrastructure

**Server**

Octopus Server allows you to use infrastructure that fits into your business's operational architecture; any infrastructure management processes and policies in place can be leveraged for your Octopus instance.  It is also your responsibility to ensure that the [infrastructure](http://g.octopushq.com/ManagingInfrastructure) performs well, is maintained and recoverable as well as scaling capabilities.

**Cloud**

Using Octopus Cloud allows you to take a step back from infrastructure management; it's our responsibility to ensure that the infrastructure is robust and scaling according to your usage. 

#### Workers

There is a distinct difference between the [Default Worker Pool](http://g.octopushq.com/OnboardingWorkersLearnMore) for Octopus Server and Octopus Cloud.  

**Server**

In Octopus Server, the [Default Worker](http://g.octopushq.com/BuiltinWorker) is the Octopus Server itself, whereas Octopus Cloud doesn't allow the Server to perform outside operations.  

When using Octopus Server, we recommend using workers where possible to reduce the resource usage on the Octopus server and, as Octopus executes under a privileged account, if we can offload work to another machine, it's wise to do so for added security.

**Cloud**

Octopus Cloud leases a [dynamic worker](http://g.octopushq.com/DynamicWorkerPools) machine from a worker pool specific to its region.

### Backups

**Cloud**

Backups are included as part of the managed service with Octopus Cloud.  The database, built-in package repository, and configuration are backed up.  If you use an external package feed, then it is your responsibility to ensure backups are configured.

**Server**

When managing your infrastructure as part of an Octopus Server installation, you can design your [backup](http://g.octopushq.com/BackupRestore) and recovery plan to match your existing infrastructure plans. 

### Monitoring

Octopus [subscriptions](http://g.octopushq.com/Subscriptions) can be used in both Octopus Cloud and Octopus Server to notify of events to watch for changes in Octopus - a change to a specific project process, a user amendment, the addition of a deployment target, the list is long!

The [REST API](http://g.octopushq.com/RestAPI) has a [reporting](http://g.octopushq.com/Reporting) endpoint that you can use to create a dashboard, available in both Octopus Cloud and Octopus Server. 

Octopus Server also offers the ability to [send logs to a central log tool](https://help.octopus.com/t/how-can-i-configure-octopus-deploy-to-write-logs-to-a-centralized-logger-such-as-seq-splunk-or-papertrail/24551) by adding a log target to the nlog config file. 

## Database

**Cloud**

The Octopus Cloud database is not accessible to customers, so all interactions with Octopus must be done using the web interface or via the API.  The great thing about this is that the database maintenance, security, and patching is all done by us!


**Server**

Octopus Server on a cloud VM gives you several options for database hosting.

- **VM** - Host SQL Server on a virtual machine.
- **PaaS** - [Google Cloud SQL](https://cloud.google.com/sql/docs/sqlserver), [Azure SQL](https://azure.microsoft.com/en-gb/services/sql-database/) or [AWS RDS for SQL Server](https://aws.amazon.com/rds/sqlserver/) for example. 
- **Docker** - Use SQL Server within a [Docker](https://www.docker.com/) container, this could be on a Windows or Linux virtual machine or a cloud container hosting platform.   

The benefit of managing your database offers more the choice of more lenient [retention policies](http://g.octopushq.com/RetentionPolicies), the ability to use a clustered database, and managing your backups.

## Security

**Cloud**

With Octopus Cloud, we secure the infrastructure and carry out penetration testing to ensure the security and integrity of the  Octopus instances.  In this case, you are responsible for:

- How you connect Octopus to your infrastructure.
- How you identify your users and control their activities within Octopus.
- How you handle sensitive information within Octopus.

**Server**

When using Octopus Server, it is your responsibility to [harden](https://octopus.com/docs/security/hardening-octopus) the Octopus instance infrastructure.  This level of control may be beneficial if there are industry-specific regulations to which you are required to adhere.  In this case, you also taking responsibility for:

- How you harden the underlying server operating system.
- How you protect the Octopus Server files on the operating system.
- How you store files generated by Octopus Server.
- How you secure your SQL Database and protect the data generated by Octopus Server.
- How you expose your Octopus Server to your infrastructure.
- How you identify your users and control their activities within Octopus.
- How you handle sensitive information within Octopus.


:::hint

When adding deployment targets and worker machines to Octopus Cloud or Octopus Server infrastructure, their security and integrity is your responsibility.

:::

### Authentication Providers

**Cloud**

Octopus Cloud [authentication](http://g.octopushq.com/AuthenticationProviders) uses [Octopus ID](http://g.octopushq.com/AuthOctopusID), allowing you to use the same account that you use to sign in at [octopus.com](https://octopus.com).  Octopus ID supports username and password or can use a Google or Microsoft account.

![octopus-id](octopus-id.png "width=500")

Octopus ID allows mapping of an Octopus user to a Google or Microsoft account however it does not allow mapping to external groups as is possible with Active Directory.

**Server**

Octopus Server has several authentication providers available to use:

- username and password
- [Active Directory](http://g.octopushq.com/AuthAD)
- [Azure Active Directory](hhttp://g.octopushq.com/AuthAzureAD)
- [Google account](http://g.octopushq.com/AuthGoogleApps)
- [Okta](http://g.octopushq.com/AuthOkta)

If desired, multiple providers can be configured concurrently. 

### IP restrictions

**Cloud**

An Octopus Cloud instance has a range of [static IP addresses](http://g.octopushq.com/CloudStaticIP) shared among customers in the same Azure region, and these can found in the Technical section of the Octopus instance management page:

![cloud-ips](cloud-ips.png "width=500")

**Server**

Using Octopus Server on a VM allows you to configure the IP and DNS to what suits your business' requirements.  You may choose to configure a load balancer to handle traffic to the Octopus web interface; this is especially useful if configuring [HA](#high-availability).

### Organize projects and environments

[Spaces](https://octopus.com/spaces) are available with both Octopus Cloud and Octopus Server; these create a hard wall between projects and infrastructure if required.

**Server**

Concurrent licenses are an additional benefit of Octopus Server.  You can use three instances for each license, run one Octopus Deploy service for production usage by your team, and set up extras for development or test. Or use two separate Octopus Deploy instances to keep production and pre-production deployments isolated.

## Networking

Connecting [Polling Tentacles](http://g.octopushq.com/PollingTentacle) over WebSockets to Octopus Cloud is not currently possible; however, we do have plans to support Polling Tentacles connecting on port 443 soon.

### Proxy support

Both Octopus Server and Octopus Cloud have support for [proxies](http://g.octopushq.com/ProxySupport).  You can specify a proxy server for Octopus to use when communicating with a Tentacle or SSH Target. You can also specify a proxy server when a Tentacle and the Octopus server make web requests to other servers.

## Region

**Server**

Using Octopus Server on a VM lets you host your Octopus instance in your choice of geographic region.  

**Cloud**

Octopus Cloud also offers you the option of hosting your instance in a region that suits you; we use Microsoft Azure to host Octopus Cloud. At the time of writing, we offer the following regions:

- West US 2
- West Europe
- Australia East

The region is selected when your first create your Octopus Cloud instance.

![cloud-region](cloud-region.png "width=500")

Should we introduce a new region that would suit your needs better, please get in touch so that we can move your instance to that region. 

### Regional Package feeds

If you operate in multiple regions and are concerned about network latency issues while transferring large packages during deployments, we'd recommend opting for an external [package feed](http://g.octopushq.com/OnboardingPackageRepositoriesLearnMore) and either replicating it across multiple regions and using DNS aliasing to have the tentacle fetch packages from geographically local package repositories.  Or, an even simpler solution than that is to have two separate but identical package repositories and have the [Feed ID as a variable](http://g.octopushq.com/DynamicPackageFeeds).  Package repositories work in this way for both **Octopus Server** and **Octopus Cloud**. 

## High Availability

[High Availability](http://g.octopushq.com/HighAvailability) (HA) can be configured for Octopus Server to enable you to run multiple Octopus Servers, distributing load and tasks between them.  

While Octopus HA is not available for Octopus Cloud, all instances are provisioned as a [Linux container running on AKS](https://octopus.com/blog/octopus-cloud-v2-why-kubernetes) (Azure's managed Kubernetes).  [Kubernetes](https://kubernetes.io/) helps us achieve the resilience and scaling flexibility to make Octopus Cloud robust and performant.

## Other considerations

### Tentacles

An Octopus tentacle machine can be configured to be either [polling](http://g.octopushq.com/PollingTentacle) or [listening](http://g.octopushq.com/ListeningTentacle).  The type of tentacles you select for your deployments will depend on your business requirements. You should also consider if the configuration of these machines has any impact on the infrastructure chosen for your Octopus instance. 

#### Tentacle firewall rules

A factor when selecting which type of tentacle to use is firewall rules.  A listening tentacle machine must have an inbound firewall rule to allow access via TCP port 10933.  The use of a polling tentacle requires no firewall rules on the tentacle, and the Octopus Server machine must allow access through the firewall inbound on TCP port 10943.  While a listening tentacle uses fewer resources, it may be that the use of a dynamic IP or where the tentacle sits behind a NAT requires a polling tentacle.

These rules should also be applied to any intermediary firewalls.

### Octopus in a container

An alternative to running Octopus Server on a virtual machine is to run it in a container.  In fact, this is how we manage our Octopus Cloud instances.  The [Windows container](http://g.octopushq.com/DockerWindows) for Octopus is fully supported, and the [Linux container](http://g.octopushq.com/DockerLinux) is, at the time of writing, part of our Early Access Program. 

## Conclusion

This post has hopefully provided clarity to your decision-making process when moving to cloud infrastructure.  Whatever your business requirements, Octopus Deploy makes release management easy and simplifies even the most complex deployments wherever you deploy your software. Runbook automation minimizes outages and gives you control over your infrastructure and applications.  

If you have any questions or would like advice on how Octopus can work for you please drop us a line at [advice@octopus.com](mailto:advice@octopus.com)

## More information

- [An explanation of worker security and scoped worker pools](http://g.octopushq.com/WorkerSecurityScopedWorkerPools)
- [Execution containers for workers](http://g.octopushq.com/ExecutionContainersForWorkers)
- [Moving octopus](http://g.octopushq.com/MovingOctopusComponents)
