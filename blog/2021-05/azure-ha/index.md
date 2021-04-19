---
title: Setting up Octopus High-Availability in Azure 
description: How to setup Octopus High-Availability in Microsoft Azure. 
author: derek.campbell@octopus.com
visibility: public
published: 2021-05-01
metaImage: 
bannerImage: 
tags:
 - Product
 - Azure
---

From the 1st September 2019, all new Octopus licenses support High Availability, meaning teams can run multiple Octopus servers, distributing load and tasks between them. We've noticed that High-Availability has become the default Octopus configuration. We've recently updated our [High-Availability](https://octopus.com/docs/administration/high-availability) documentation to give people more information and options on where to host Highly-Available Octopus Deploy. In this blog, I set up Octopus High-Availability on Azure and expand on the possibilities for hosting Highly Available Octopus on [Microsoft Azure](https://azure.microsoft.com/en-us/).

!toc

## Why should you make Octopus Highly-Available?

[High Availability](https://octopus.com/docs/administration/high-availability) enables you to run multiple Octopus Deploy Servers, distributing load and tasks between them. High Availability has several benefits, which include:

- Higher resilience for business-critical workloads
- Simplifies maintenance tasks such as [server patching](https://en.wikipedia.org/wiki/Patch_(computing))
- Performance and Scalability
- No one enjoys downtime

## Octopus High-Availability Components

An Octopus: HA configuration requires four main components:

- **A load balancer**
  Load Balancers direct user traffic bound for the Octopus web interface between the different Octopus Server nodes.
- **Octopus Server nodes**
  These run the Octopus Server windows service. They serve user traffic and orchestrate deployments.
- **A database**
  Most data used by the Octopus Server nodes are stored in this Database.
- **Shared storage**
  Some larger files - like [NuGet packages](https://octopus.com/docs/packaging-applications/package-repositories/index.md), artifacts, and deployment task logs - aren't suitable to be stored in the Database and so must be stored in a shared folder available to all nodes.

## Octopus Virtual Machines

When creating a Highly-Available configuration, you will need to spin up a minimum of two Virtual Machines in Azure to host Octopus. We don't have a one-size-fits-all spec for Octopus as it will depend on:

- [Number and type of Deployment Targets](https://octopus.com/docs/administration/retention-policies/)
- [Retention Policies](https://octopus.com/docs/administration/retention-policies/)
- [Number of concurrent tasks](https://octopus.com/docs/support/increase-the-octopus-server-task-cap/)

If you have a reasonably small workload in Octopus, you can probably go for a smaller Virtual Machine. Still, the Azure D Series Virtual Machines are a great place to start as they are for general purpose and fit most scenarios reasonably well. Our recommendation would be to consider your workload and then use one of the D Series VM's and see how well this performs for your requirements.

In this instance, I spun up two Virtual Machines, one call **Octo1** and **Octo2** using **D2s V2**, which has two vCPU and 8GB of memory using **Server 2019**. This specification is a great place to start, and you can change the D Series quite easily to other sizes. So, in theory, you can increase resources and decrease resources as required. You could use some form of automation here to scale horizontally or vertically.

### Virtual Machine Disks

You'll need to consider what type of storage you want for your Octopus Virtual Machines, and you can see a full list of [Availability Disk Types](https://docs.microsoft.com/en-us/azure/virtual-machines/disks-types), and it compares them. There are a few to consider:

- [Ultra Disk](https://docs.microsoft.com/en-us/azure/virtual-machines/disks-types#ultra-disk)
- [Premium SSD](https://docs.microsoft.com/en-us/azure/virtual-machines/disks-types#premium-ssd)
- [Standard SSD](https://docs.microsoft.com/en-us/azure/virtual-machines/disks-types#standard-ssd)
- [Standard HDD](https://docs.microsoft.com/en-us/azure/virtual-machines/disks-types#standard-hdd)

The critical bit is to remember that this is only for the VM, and I selected **Standard SSD** for my purposes. The main reason for this was that the cost and performance matched my requirements. Octopus isn't very disk-intensive, which means you're unlikely to get many benefits using Ultra Disks. Still, you should consider Premium SSD if you're using Octopus with thousands of projects, as this can be beneficial.

### Azure Availability Sets vs. Azure Availability Zones

Please check out the [Azure Docs] for a complete list of availability options for Azure Virtual Machines; please check out the [Azure Docs](https://docs.microsoft.com/en-us/azure/virtual-machines/availability) as we won't cover all of these.

- [Azure Availability Zones](https://docs.microsoft.com/en-us/azure/availability-zones/az-overview#availability-zones) are separate data centers within Azure Region, with dedicated power, cooling, and networking. With this option, when using Availability Zones, you are ensuring Octopus remains resilient to failure in your primary Azure Region. To ensure resiliency, there's a minimum of three separate zones in all enabled regions. Azure offers a 99.99% uptime SLA for this option.
- [Azure Availability Sets](https://docs.microsoft.com/en-us/azure/virtual-machines/availability-set-overview) is a logical grouping of VMs that provides redundancy and Availability. Azure offers a 99.95% SLA for Availability Sets, and there are no costs for this, apart from the Virtual Machine Costs.

When designing and configuring my Octopus on Microsoft Azure, I went for the Azure Availability Zones option purely for the increased SLA, and it's also what Microsoft is generally recommending for High Availability. I set up my two Virtual Machines **Octo1** and **Octo2** in **Availability Zone 1** and **Availability Zone 2**. This gives me tolerance in Octopus HA as it's using different logical Data Centers but also benefits of having low-latency access to the storage and the SQL database.

Most Octopus High-Availability will contain two Virtual Machines but using three or more, you will need to place them into their Zones.

An example configuration for multi-VM Octopus High-Availability would be:

- Octo1 in AZ1, Octo2 in AZ2
- Octo1 in AZ1, Octo2 in AZ2, Octo3 in AZ3
- Octo1 in AZ1, Octo2 in AZ2, Octo3 in AZ3, Octo4 in AZ1

We have only tested Octopus to 8 nodes, but you can separate these over as many of the Availability Zones as you wish.

## SQL Database

Octopus is underpinned by a SQL Database that stores environments, projects, variables, releases, and deployment history. You will need to spin up a SQL server in Azure, and there are two options that you should consider, and these are:

- [SQL Server on a Virtual Machine](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-iaas-overview/)
- [Azure SQL Database as a Service](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-technical-overview/)

Octopus natively works with both of these options, and we don't have a preference and will be a decision you will need to make. If you have access to a Database Administrator, then I would seek out their expertise on the matter as they may provide further insight.

### SQL Virtual Machine vs. Azure SQL

I still see that most organizations are using Virtual Machines when they are provisioning their SQL workload in the cloud, and in this section, I will go through the benefits of picking SQL VM's and some of the drawbacks.

As we are after a highly available configuration, Octopus, we need to factor in high Availability at the SQL level. What this means is that you are going to need a minimum of 2 SQL servers, preferably in a [SQL cluster in Azure](https://techcommunity.microsoft.com/t5/Premier-Field-Engineering/Configure-SQL-Server-Failover-Cluster-Instance-on-Azure-Virtual/ba-p/371464), or an [Always On Availability Group](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-availability-group-tutorial) in Azure. For the most part, I will keep this part high-level as there is a lot of really great content on these topics out there already, and you may have a Database Administrator who will carry this out for you. If you have this already on Azure, I would recommend using that setup to host Octopus, preferably on a dedicated SQL instance.

There are several benefits of using SQL Virtual Machines over Azure SQL, and these are:

- Greater flexibility
- More control
- Hosting multiple Databases without additional cost.

Some of the drawbacks of using SQL Virtual Machines over Azure SQL:

- Higher Total Cost of Ownership
- Increased Setup time
- Maintaining Infrastructure and Database(s)

I spend a considerable amount of time doing Proof of Concepts, and I am a big fan of anything [PaaS](https://en.wikipedia.org/wiki/Platform_as_a_service). I particularly like [Azure SQL Database Service](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-technical-overview), as I don't need to invest a considerable amount of time spinning up Virtual Machines's, Network Security Groups, Configuring SQL, Firewall rules, Maintenance plans, and so forth. I can log on to the [Azure Portal](https://portal.azure.com/) and spin up a new SQL Server, Database, and connection strings in a matter of minutes, and if you have [ARM Templates](https://azure.microsoft.com/en-gb/resources/templates/) in place for this, it can take a few moments. [Infrastructure as Code](https://en.wikipedia.org/wiki/Infrastructure_as_code) is a huge time saver, but I realize that this may not be for everyone's preference just yet.

When spinning up a database on Azure SQL, I can create a geo-replicated or locally-replicated database in a few moments, and that's my Database highly available. I need to configure the Connection string for Octopus, and I am good to go.

There are many benefits of using Azure SQL over SQL Virtual Machines:

- It's easier to configure
- Spin up and tear down as you require in seconds/minutes
- Making the Database highly available is a couple of commands or clicks away.
- Managed Backup and Maintenance tasks.
- Great Azure AQL Analytics and Monitoring built-in.

Some of the drawbacks of using Azure SQL over SQL Virtual Machines:

- Less control
- Restoring a backup when something goes wrong takes considerably longer.
- Refactoring SQL scripts
- Trying to understand what a [DTU](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-service-tiers-dtu) is.

As you can see with both options, they have their merits and drawbacks, and it's probably best if you give this some thought and pick the right solution for you and your Organizational needs.

### SQL Database Selection

In my example, I selected:

- Azure SQL Server in West Europe as this is my primary region for all
- Azure SQL Server in North Europe for my Geo-Replicated Octopus Database for Disaster Recovery needs.
- Created a database named **Octo-HA** in the primary region and SQL Server
- Browsed to Geo-Replication on the Primary Azure SQL Database and enabled Replication
- Let it replicate to the secondary database server and region

At this point, you have a primary Database Server and Database, and it's synced cross-region to a secondary server and Database.

:::info
Zone Redundant Databases are currently in Preview in Azure. Had this been available, I would have used Zone Redundant Databases as this is the same level of redundancy I have set for my Azure Virtual Machines.
:::

### SQL Database Specifications

This part of the blog covers options for SQL Server Performance. I'd recommend checking out [High Availability SLA](https://docs.microsoft.com/en-us/azure/azure-sql/database/high-availability-sla) doc to ensure you pick the correct sizing of the Database and Server.

In my example, I selected:

- General Purpose
- Provisioned, which provides Computer resources that are pre-allocated and billed per hour.
- 2 vCores
- Max Data Size of 30GB
- Zone Redundancy turned on. (Zone Redundancy adds about 20% to the cost)

These specifications would be a great place to start from in a Highly-Available configuration, but you may need to consider your workload, as this may be far too big for your requirements if you own a small Octopus instance.

If you own a large instance, I'd consider the HyperScale and Business Critical loads in Microsoft Azure. Try and find the right fit for your requirements for SQL, as you don't want a slow-performing Octopus instance, but you probably also don't want to size it too big and pay too much for your Database hosting.

## Storage

In a single node setup, you would typically host Octopus on [Local Storage](https://en.wikipedia.org/wiki/Local_storage) on either `C:\Octopus` or `D:\Octopus.` You're going to need some local storage for Octopus unless you decide to present an Azure File Share as a mapped drive or as a Symbolic link to the server. Our recommendation is to host your logs and configuration of Octopus locally on the server. It avoids any potential issues with accidentally pointing all of your Octopus node's logs location to the same file, which would cause file locking issues and cause the Octopus server to stop until you resolved it.

### Artifacts, Packages & Task Logs

Octopus stores several files that are not suitable to store in the Database. These include:

- NuGet packages used by the [built-in NuGet repository](https://octopus.com/docs/packaging-applications/package-repositories) inside Octopus. These packages can often be substantial.
- Artifacts collected during a deployment. Teams using Octopus sometimes use this feature to collect large log files and other files from machines during a deployment.
- Task logs are text files that store all of the log output from deployments and other tasks.

As with the Database, you'll tell the Octopus Servers where to store them as a file path within your operating system from the Octopus perspective. Octopus doesn't care what technology you use to present the shared storage; it could be a mapped network drive or a UNC path to a file share. Each of these three types of data is stored in a different place.

Whichever way you provide the shared storage, a few considerations to keep in mind:

- To Octopus, it needs to appear as a mapped network drive (e.g., `D:\`) or a UNC path to a file share (e.g., \\server\path)
- The service account that Octopus runs as needs full control over the directory
- Drives are mapped per-user, so you should assign the drive using the same service account that Octopus is running under

### Azure Files

If your Octopus Server is running in Microsoft Azure, there is only one solution unless you have a [DFS Replica](https://docs.microsoft.com/en-us/windows-server/storage/dfs-replication/dfsr-overview) in Azure. That solution is [Azure File Storage](https://docs.microsoft.com/en-us/azure/storage/files/storage-files-introduction) presents a file share over SMB 3.0 that will is shared across all of your Octopus servers.

Once you have created your File Share, I find the best option is to add the Azure File Share as a [symbolic link](https://en.wikipedia.org/wiki/Symbolic_link) and then adding this to `C:\Octopus\` for the Artifacts, Packages, and TaskLogs which need to be available to all nodes.

Run the below **before installing Octopus**.

````powershell
# Add the Authentication for the symbolic links. You can get this from the Azure Portal.

cmdkey /add:octostorage.file.core.windows.net /user:Azure\octostorage /pass:XXXXXXXXXXXXXX

# Add Octopus folder to add symbolic links

New-Item -ItemType directory -Path C:\Octopus
New-Item -ItemType directory -Path C:\Octopus\Artifacts
New-Item -ItemType directory -Path C:\Octopus\Packages
New-Item -ItemType directory -Path C:\Octopus\TaskLogs

# Add the Symbolic Links. Do this before installing Octopus.

mklink /D C:\Octopus\TaskLogs \\octostorage.file.core.windows.net\octoha\TaskLogs
mklink /D C:\Octopus\Artifacts \\octostorage.file.core.windows.net\octoha\Artifacts
mklink /D C:\Octopus\Packages \\octostorage.file.core.windows.net\octoha\Packages
````

[Install Octopus](https://octopus.com/docs/installation) and then run the below.

````powershell
# Set the path
& 'C:\Program Files\Octopus Deploy\Octopus\Octopus.Server.exe' path --artifacts "C:\Octopus\Artifacts"
& 'C:\Program Files\Octopus Deploy\Octopus\Octopus.Server.exe' path --taskLogs "C:\Octopus\TaskLogs"
& 'C:\Program Files\Octopus Deploy\Octopus\Octopus.Server.exe' path --nugetRepository "C:\Octopus\Packages"
````

## Load Balancing

When you configured the first Octopus Server node and each of the subsequent nodes, you would have configured the HTTP endpoint that the Octopus web interface is available on. The final step is to configure a load balancer to direct user traffic between each of the Octopus Server nodes.

Octopus can work with any load balancer technology, including hardware and software load balancers. In Azure, we have the following Load Balancers available to us.

- [Azure Traffic Manager](https://docs.microsoft.com/en-us/azure/traffic-manager/traffic-manager-overview)
- [Azure Application Gateway](https://docs.microsoft.com/en-us/azure/application-gateway/overview)
- [Azure Load Balancer](https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-overview)
- [Azure Front Door](https://docs.microsoft.com/en-us/azure/frontdoor/front-door-overview)
- [Kemp LoadMaster](https://kemptechnologies.com/uk/solutions/microsoft-load-balancing/loadmaster-azure/)
- [F5 Big-IP Virtual Edition](https://www.f5.com/partners/technology-alliances/microsoft-azure)

After evaluating the options on Azure, my preference is the Azure Load Balancer option as this is a feature-rich Load Balancer that meets my requirements. If you want to compare all of the Azure options for Load Balancing, check out [Choose a Load Balancing service](https://docs.microsoft.com/en-us/azure/architecture/guide/technology-choices/load-balancing-overview)

:::tip
Create your Azure Load Balancer before you create the Virtual Machines, as you can select the Load Balancer during provisioning.
:::

### Load balancer session persistence

We typically recommend using a round-robin (or similar) approach for sharing traffic between the nodes in your cluster, as the Octopus Web Portal is stateless.

However, each node in the cluster keeps a local cache of data, including user permissions. There is a known issue that occurs when a user's permissions change. The local cache is only invalidated on the node where the change was made.

To work around this issue in the meantime, you can configure your load balancer with **session persistence**. This will ensure user sessions are routed to the same node.

## Authentication Providers

If you're migrating from On-Premises to Azure, you will need to consider your Authentication Providers. You're likely using **Active Directory** on-Premises, and generally, this isn't something that's supported in Octopus in Azure. The main reason for this is that you need a Domain Controller in the same network as your Octopus installation to allow for Authentication of your users using Active Directory.

If you have Domain Controllers in Azure that are contactable, you can continue to use Active Directory authentication.

If you do not have Domain Controllers that are contactable in Azure, then you'd need to consider switching to:

- [Azure Active Directory](https://octopus.com/docs/security/authentication/azure-ad-authentication)
- [GoogleApps](https://octopus.com/docs/security/authentication/googleapps-authentication)
- [Okta](https://octopus.com/docs/security/authentication/okta-authentication)
- [Built-in Users and Teams](https://octopus.com/docs/security/users-and-teams)

### Moving Authentication Providers

If you have used Active Directory on-Premises and you are moving to Azure, and you can't continue using Active Directory, you can associate multiple external identities to a single user in Octopus. The most common migration is likely to be from Active Directory to Azure Active Directory. In this example, assuming I have a user named **Derek.Campbell** on a domain called **work.local** and an Active Directory tenant of **worklocal.onmicrosoft.com** I would:

- Setup [Azure Active Directory](https://octopus.com/docs/security/authentication/azure-ad-authentication)
- Add my **Derek.Campbell@Worklocal.OnMicrosoft.com** account to the Octopus Deploy user that also has my **Derek.Campbell@Work.local** user.
- Remove **Derek.Campbell@work.local** from the user.
- Test Authentication
- Rinse and Repeat for all users.

:::info
Please check out [this script](https://github.com/OctopusDeploy/OctopusDeploy-Api/blob/master/REST/PowerShell/Users/AddAzureActiveDirectoryLoginToUsers.ps1) as this is likely to help with any migrations from On-Premises to Azure.
:::

## Networking

Networking is a contentious issue at the best of times, and your configuration will depend heavily on your existing network topology and standards. I'd consider implementing one or some of the below to help protect your Azure workload:

- [Azure Bastion](https://azure.microsoft.com/en-gb/services/azure-bastion/)
- [VPN Gateway](https://azure.microsoft.com/en-gb/services/vpn-gateway/)
- [ExpressRoute](https://azure.microsoft.com/en-gb/services/expressroute/)
- [A Jump Box](https://en.wikipedia.org/wiki/Jump_server)

I'd look to use existing methods to connect to Azure if you already have this in place. If you have ExpressRoute, then this is the best approach, but much like with anything, it's the most expensive. If you have a VPN Gateway, a jump box, or even Azure Bastion service, I would recommend using these to your advantage.

The biggest recommendation I can make here is to reduce your [surface of attack](https://en.wikipedia.org/wiki/Attack_surface) while keeping your networking as straightforward as you can without causing any potential security issues.

- Where possible, try and use [Internal IP's and networks](https://en.wikipedia.org/wiki/Private_network) over Public IPs, particularly for your SQL configuration.
- Use a VPN or a Jump/Bastion box. Preferably both.
- Secure Octopus to use HTTPS only with a valid certificate.

### Polling Tentacles

Listening Tentacles require no special configuration for High Availability.  Polling Tentacles, however, poll a server at regular intervals to check if any tasks are waiting for the Tentacle to perform. In a High Availability scenario, Polling Tentacles must poll all of the Octopus Servers in your configuration. You could poll a load balancer, but there is a risk, depending on your load balancer configuration, that the Tentacle will not poll all servers promptly.  You could also configure the Tentacle to poll each server by registering it with one of your Octopus Servers and then adding each Octopus Server to the Tentacle.config file. There are two options to add Octopus Servers, via the command line or via editing the Tentacle.config file directly:

**Tentacle.config**

Configuring the Tentacle via the command line is the preferred option with the command executed once per server; an example command using the default instance can be seen below:

```powershell
C:\Program Files\Octopus Deploy\Tentacle>Tentacle poll-server --server=http://my.Octopus.server --apikey=API-77751F90F9EEDCEE0C0CD84F7A3CC726AD123FA6
```

For more information on this command, please refer to the [Tentacle Poll Server options document](https://octopus.com/docs/octopus-rest-api/tentacle.exe-command-line/poll-server.md)

Alternatively, you can edit Tentacle.config directly to add each Octopus Server (this is interpreted as a JSON array of servers). This method is not recommended as the Octopus service for each server will need to be restarted to accept incoming connections via this method.

```xml
<set key="Tentacle.Communication.TrustedOctopusServers">
[
  {"Thumbprint":"77751F90F9EEDCEE0C0CD84F7A3CC726AD123FA6","CommunicationStyle":2,"Address":"https://10.0.255.160:10943","Squid":null,"SubscriptionId":"poll://g3662re9njtelsyfhm7t/"},
  {"Thumbprint":"77751F90F9EEDCEE0C0CD84F7A3CC726AD123FA6","CommunicationStyle":2,"Address":"https://10.0.255.161:10943","Squid":null,"SubscriptionId":"poll://g3662re9njtelsyfhm7t/"},
  {"Thumbprint":"77751F90F9EEDCEE0C0CD84F7A3CC726AD123FA6","CommunicationStyle":2,"Address":"https://10.0.255.162:10943","Squid":null,"SubscriptionId":"poll://g3662re9njtelsyfhm7t/"}
]
</set>
```

Notice there is an address entry for each Octopus Server in the High Availability configuration. Depending on your configuration and Network topology, you can use the Private IP or the Public IP and/or FQDN for each Octopus node. An example below shows what it would be like to register to **octo1.domain.com**, **octo2.domain.com**, and **octo3.domain.com** on port **10943**.

```xml
<set key="Tentacle.Communication.TrustedOctopusServers">
[
  {"Thumbprint":"77751F90F9EEDCEE0C0CD84F7A3CC726AD123FA6","CommunicationStyle":2,"Address":"https://octo1.domain.com:10943","Squid":null,"SubscriptionId":"poll://g3662re9njtelsyfhm7t/"},
  {"Thumbprint":"77751F90F9EEDCEE0C0CD84F7A3CC726AD123FA6","CommunicationStyle":2,"Address":"https://octo2.domain.com:10943","Squid":null,"SubscriptionId":"poll://g3662re9njtelsyfhm7t/"},
  {"Thumbprint":"77751F90F9EEDCEE0C0CD84F7A3CC726AD123FA6","CommunicationStyle":2,"Address":"https://octo3.domain.com:10943","Squid":null,"SubscriptionId":"poll://g3662re9njtelsyfhm7t/"}
]
</set>
```

In the below example, I use the public IP instead of an FQDN or a private IP.

```xml
<set key="Tentacle.Communication.TrustedOctopusServers">
[
  {"Thumbprint":"77751F90F9EEDCEE0C0CD84F7A3CC726AD123FA6","CommunicationStyle":2,"Address":"https://1.2.3.4:10943","Squid":null,"SubscriptionId":"poll://g3662re9njtelsyfhm7t/"},
  {"Thumbprint":"77751F90F9EEDCEE0C0CD84F7A3CC726AD123FA6","CommunicationStyle":2,"Address":"https://1.2.3.5:10943","Squid":null,"SubscriptionId":"poll://g3662re9njtelsyfhm7t/"},
  {"Thumbprint":"77751F90F9EEDCEE0C0CD84F7A3CC726AD123FA6","CommunicationStyle":2,"Address":"https://1.2.3.6:10943","Squid":null,"SubscriptionId":"poll://g3662re9njtelsyfhm7t/"}
]
</set>
```

## Migration

If you're migrating an instance of Octopus to Azure, our recommended approach is:

- Set up the Infrastructure
- Setup the Octopus folders and storage location
- Run through highly-available Octopus installation
- Do a Proof of Concept move to the new instance
- Test Authentication & Deployments
- Plan a Production move when confirmed.
- Plan downtime
- Migrate using [Moving the Octopus Server and Database](https://octopus.com/docs/administration/managing-infrastructure/moving-your-octopus/move-the-database-and-server)
- Switch to the new Azure setup

## Summary

As you can see, as much as high Availability in Azure is straightforward. There is a lot to consider when moving from On-Premises to Azure hosting for Octopus Deploy. In this blog, I've explained and recommended some technologies to use to help you when you're setting Octopus in High-Available on Microsoft Azure.

I hope that these tips will help you setup Highly-Available Octopus on Microsoft Azure!

Happy Deployments!
