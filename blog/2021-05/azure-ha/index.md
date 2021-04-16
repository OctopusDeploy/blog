---
title: Setting up Octopus High-Availability in Azure 
description: How to setup Octopus High-Availability in Microsoft Azure. 
author: derek.campbell@octopus.com
visibility: public
published: 2021-05-01
metaImage: image.png
bannerImage: image.png
tags:
 - Product
---

**TODO - Update image reference when received from Design**
![Image details](image.png)

All new Octopus licenses, from the 1st September 2019 support High Availability, meaning teams can run multiple Octopus servers, distributing load and tasks between them. We've noticed that High-Availability has become the default Octopus configuration and we've recently updated our [High-Availability](https://octopus.com/docs/administration/high-availability) documentation to give people more choice on where to host Highly-Available Octopus Deploy. In this blog, I expand on the options for hosting HA Octopus on [Microsoft Azure](https://azure.microsoft.com/en-us/).

!toc

## Octopus High-Availability Components

An Octopus: HA configuration requires four main components:

- **A load balancer**
  This will direct user traffic bound for the Octopus web interface between the different Octopus Server nodes.
- **Octopus Server nodes**
  These run the Octopus Server windows service. They serve user traffic and orchestrate deployments.
- **A database**
  Most data used by the Octopus Server nodes is stored in this database.
- **Shared storage**
  Some larger files - like [NuGet packages](/docs/packaging-applications/package-repositories/index.md), artifacts, and deployment task logs - aren't suitable to be stored in the database, and so must be stored in a shared folder available to all nodes.

## Why should you make Octopus Highly-Available?

[High Availability](https://octopus.com/docs/administration/high-availability) enables you to run multiple Octopus Deploy Servers, distributing load and tasks between them. High Availability has several benefits which include:

- Higher resilience for business-critical workloads
- Simplifies maintenance tasks such as [server patching](https://en.wikipedia.org/wiki/Patch_(computing))
- Performance benefits
- Reducing cost

### Octopus Virtual Machines

When creating a Highly-Available configuration, you are going to need to spin up a minimum of two Virtual Machines in Azure to host Octopus. We don't have a one-size-fits-all spec for Octopus as it will depend on:

- [Number and type of Deployment Targets](https://octopus.com/docs/administration/retention-policies/)
- [Retention Policies](https://octopus.com/docs/administration/retention-policies/)
- [Number of concurrent tasks](https://octopus.com/docs/support/increase-the-octopus-server-task-cap/)

If you have a reasonably small workload in Octopus, then you can probably go for a smaller Virtual Machine. Still, the Azure D Series Virtual Machines are a great place to start as they are for general purpose and fit most scenarios reasonably well. Our recommendation would be to consider your workload and then use one of the D Series VM's and see how well this performs for your requirements.

### SQL Database

Octopus is underpinned by a SQL Database that stores environments, projects, variables, releases, and deployment history. You will need to spin up a SQL server in Azure, and there are two options which you should consider, and these are:

- [SQL Server on a Virtual Machine](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-iaas-overview/)
- [Azure SQL Database as a Service](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-technical-overview/)

Octopus natively works with both of these options, and we don't have a preference and will be a decision you will need to make. If you have access to a Database Administrator, then I would seek out their expertise on the matter as they may be able to provide further insight.

### SQL Virtual Machine vs. Azure SQL

I still see that most organizations are using Virtual Machines for when they are provisioning their SQL workload in the cloud, and in this section, I will go through the benefits of picking SQL VM's and some of the drawbacks.

As we are after a highly-available configuration Octopus, then we need to factor in high-availability at the SQL level. What this means is that you are going to need a minimum of 2 SQL servers, preferably in a [SQL cluster in Azure](https://techcommunity.microsoft.com/t5/Premier-Field-Engineering/Configure-SQL-Server-Failover-Cluster-Instance-on-Azure-Virtual/ba-p/371464), or an [Always On Availability Group](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-availability-group-tutorial) in Azure. For the most part, I am going to keep this part high-level as there is a lot of really great content on these topics out there already, and you may have a Database Administrator, who will carry this out for you. If you have this already on Azure, then I would recommend using that setup to host Octopus, preferably on a dedicated SQL instance.

There are several benefits of using SQL Virtual Machines over Azure SQL, and these are:

- Greater flexibility
- More control
- Hosting multiple Databases without additional cost.

Some of the drawbacks of using SQL Virtual Machines over Azure SQL:

- Higher Total Cost of Ownership
- Increased Setup time
- Maintaining Infrastructure and Database(s)

I spend a considerable amount of time doing Proof of Concepts, and I am a big fan of anything [PaaS](https://en.wikipedia.org/wiki/Platform_as_a_service). I particularly like [Azure SQL Database Service](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-technical-overview), as I don't need to invest a considerable amount of time spinning up Virtual Machines's, Network Security Groups, Configuring SQL, Firewall rules, Maintenance plans and so forth. I can log on to the [Azure Portal](https://portal.azure.com/) and spin up a new SQL Server, Database, and connection strings in a matter of minutes and if you have [ARM Templates](https://azure.microsoft.com/en-gb/resources/templates/) in place for this, it can take seconds. [Infrastructure as Code](https://en.wikipedia.org/wiki/Infrastructure_as_code) is a huge time saver, but I realize that this may not be for everyone's preference just yet.

When spinning up a database on Azure SQL, I can create a geo-replicated or locally-replicated database in a few moments, and that's my DatabaseDatabase highly available. All I need to do now is configure the Connection string for Octopus, and I am good to go.

There are many benefits of using Azure SQL over SQL Virtual Machines:

- Easier to configure
- Spin up and tear down as you require in seconds/minutes
- Making the database highly-available is a couple of commands or clicks away.
- Managed Backup and Maintenance tasks.
- Great Azure AQL Analytics and Monitoring built-in.

Some of the drawbacks of using Azure SQL over SQL Virtual Machines:

- Less control
- Restoring Backups when something goes wrong, takes considerably longer.
- Refactoring SQL scripts
- Trying to understand what a [DTU](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-service-tiers-dtu) is.

As you can see with both options, they have their merits and their drawbacks, and it's probably best if you give this some thought and pick the right solution for you and your Organizational needs.

### Storage

In a single node setup, you would typically host Octopus on [Local Storage](https://en.wikipedia.org/wiki/Local_storage) on either `C:\Octopus` or `D:\Octopus`. You're going to need some local storage for Octopus unless you decide to present an Azure File Share as a mapped drive or as a Symbolic link to the server. Our recommendation is to host your logs and configuration of Octopus locally on the server. It avoids any potential issues with accidentally pointing all of your Octopus nodes logs location to the same file, which would cause file locking issues and cause the Octopus server to stop until you resolved it.

### Artifacts, Packages & Task Logs

Octopus stores several files that are not suitable to store in the Database. These include:

- NuGet packages used by the [built-in NuGet repository](https://octopus.com/docs/packaging-applications/package-repositories) inside Octopus. These packages can often be substantial.
- Artifacts collected during a deployment. Teams using Octopus sometimes use this feature to collect large log files and other files from machines during a deployment.
- Task logs, which are text files that store all of the log output from deployments and other tasks.

As with the Database, from the Octopus perspective, you'll tell the Octopus Servers where to store them as a file path within your operating system. Octopus doesn't care what technology you use to present the shared storage; it could be a mapped network drive or a UNC path to a file share. Each of these three types of data is stored in a different place.

Whichever way you provide the shared storage, a few considerations to keep in mind:

- To Octopus, it needs to appear as a mapped network drive (e.g., `D:\`) or a UNC path to a file share (e.g., \\server\path)
- The service account that Octopus runs as needs full control over the directory
- Drives are mapped per-user, so you should assign the drive using the same service account that Octopus is running under

### Azure Files

If your Octopus Server is running in Microsoft Azure, there is only one solution unless you have a [DFS Replica](https://docs.microsoft.com/en-us/windows-server/storage/dfs-replication/dfsr-overview) in Azure. That solution is [Azure File Storage](https://docs.microsoft.com/en-us/azure/storage/files/storage-files-introduction) - it just presents a file share over SMB 3.0 that will is shared across all of your Octopus servers.

Once you have created your File Share, I find the best option is to add the Azure File Share as a [symbolic link](https://en.wikipedia.org/wiki/Symbolic_link) and then adding this in to `C:\Octopus\` for the Artifacts, Packages, and TaskLogs which need to be available to all nodes.

Run the below before installing Octopus.

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

### Load Balancer

This one took me a bit of time to make a selection as there are a few options for balancing your loads in Azure. It will depend entirely on your preference and what your Network team prefers, as are many options, and we are only listing a few of these below.

- [Azure Traffic Manager](https://docs.microsoft.com/en-us/azure/traffic-manager/traffic-manager-overview)
- [Azure Application Gateway](https://docs.microsoft.com/en-us/azure/application-gateway/overview)
- [Azure Load Balancer](https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-overview)
- [Kemp LoadMaster](https://kemptechnologies.com/uk/solutions/microsoft-load-balancing/loadmaster-azure/)
- [F5 Big-IP Virtual Edition](https://www.f5.com/partners/technology-alliances/microsoft-azure)

My preference after evaluating the options on Azure is the Azure Load Balancer option as this is a feature-rich

### Networking

Networking at the best of times is a contentious issue, and your configuration will depend heavily on your existing network topology and standards. I'd consider implementing one or some of the below to help protect your Azure workload:

- [Azure Bastion](https://azure.microsoft.com/en-gb/services/azure-bastion/)
- [VPN Gateway](https://azure.microsoft.com/en-gb/services/vpn-gateway/)
- [ExpressRoute](https://azure.microsoft.com/en-gb/services/expressroute/)
- [A Jump Box](https://en.wikipedia.org/wiki/Jump_server)

I'd look to use existing methods to connect to Azure if you already have this in place. If you have ExpressRoute, then this is the best approach, but much like with anything, it's the most expensive. If you have a VPN Gateway, or a jump box or even Azure Bastion service, then I would recommend using these to your advantage.

The biggest recommendation I can make here is to reduce your [surface of attack](https://en.wikipedia.org/wiki/Attack_surface) while keeping your networking as straight forward as you can without causing any potential security issues.

- Where possible, try and use [Internal IP's and networks](https://en.wikipedia.org/wiki/Private_network) over Public IP's, particularly for your SQL configuration.
- Use a VPN or a Jump/Bastion box. Preferably both.
- Secure Octopus to use HTTPS only with a valid certificate.

## Summary

As you can see, there is quite a lot to consider when moving from On-Premise to Azure.
