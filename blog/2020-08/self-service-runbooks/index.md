---
title: Self-service runbooks
description: Learn to provide common adminstrative tasks as self-service runbooks
author: shawn.sesna@octopus.com
visibility: private
published: 2999-01-01
metaImage:
bannerImage:
tags:
 - 
---

Breaking down the barriers between developers and operations is the cornerstone of the DevOps philosophy.  Developers want to be able to deliver their code quickly and safely and need the ability to perform common administrative tasks which are often firmly in the realm of operations.  Security, auditing, and knowledge of how to properly perform the operation are the most common barriers which prevent a developer from having the ability to do these tasks.  In this post, I'll go over some examples of how you can use [runbooks](https://octopus.com/docs/runbooks) to empower developers to do tasks that would enable them to move faster without granting additional permissions and auditing the activity.

## Auditing
Out of the box, Octopus Deploy provides a robust auditing mechanism for deployments, capturing who did what and when.  While runbooks work in a different manner than a deployment, it audits the run of a runbook in much the same way a deployment does.  This makes it easy for auditors to see who executed the runbook.

## Examples of self-service tasks
Below is a list of some of the types of activities that could be implemented for self-service.  This is my no means an exhaustive list, it merely provides you a starting point of what is possible.

### Restarting web applications
Developers typically have elevated or even administrator rights to their development environment.  Once their application has been deployed to a server, permissions are usually restricted to simulate what a production-like environment would be like.  IIS has the ability to grant [remote administrative permissions](https://docs.microsoft.com/en-us/iis/manage/remote-administration/remote-administration-for-iis-manager), however, this is to the whole IIS instance and not granular to a specific site/application.  

Using a runbook, you could create project specific processes that only start/stop the applications that are related to the specific Octopus project.
- [Example for IIS](https://octopus.com/docs/runbooks/runbook-examples/routine/iis-maintenance)
- [Example for Tomcat](https://octopus.com/docs/runbooks/runbook-examples/routine/restarting-tomcat)

### Restarting services
Testing sometimes finds bugs that lead to services becomming unresponsive.  The ability to restart a service on an Operating System (OS) usually requires a rather high level of permission.  Using a runbook, it is possible to give a developer the ability to restart a service on a server without having any additional permissions assigned to them.  This could help elminate the need to submit a support ticket and wait for an operations staff perform the task for them.
- [Example for Windows](https://octopus.com/docs/runbooks/runbook-examples/services/windows-services)
- [Example for Ubuntu](https://octopus.com/docs/runbooks/runbook-examples/services/restart-linux-service)

### Backing up a database
Testing database updates is usually a one way trip.  Unless you execute everything within the same transaction and roll it back, the changes are permenant and often difficult to revert.  The path of least resistance is to back up the database before starting so you have something to go back to.  Backup operations on database servers often require elevated database server permissions.  Database Administrators (DBA) usually have a specific method that is employed to perform backups, sometimes involving third-party tools to help manage it.  Using a runbook, the DBA could create or at least provide input into how a database backup should be done and give the developer the ability to backup the database whenever they need to.
- [Example for SQL Server](https://octopus.com/docs/runbooks/runbook-examples/databases/backup-mssql-database)
- [Example for MySQL](TBD)

### Restoring a database
Along with the ability to backup a database, having the ability to restore a database without needing to wait for a DBA or filling out a support ticket could drastically reduce development lead times.  Using a runbook, you could not only restore a backup taken previously, you could also provide a method for restoring a database from a different environment, such as restoring a production copy to test.
- [Example for SQL Server](https://octopus.com/docs/runbooks/runbook-examples/databases/restore-mssql-database)
- [Example for SQL Server, restoring to different environment](https://octopus.com/docs/runbooks/runbook-examples/databases/restore-mssql-database-to-environment)

### Provisioning entire environments for feature branch development
In a previous blog post, we talked about [feature branching](https://octopus.com/blog/rethinking-feature-branch-deployments).  Within this post, we spoke about implementing dynamic environments whenever a new feature branch is created, then tearing it down when the branch is deleted.  This type of self-service allows the developers to provision production-like environments to test their code on, then remove them when they're no longer necessary.
- [Example of deploying Azure ARM](https://octopus.com/docs/runbooks/runbook-examples/azure/resource-groups)
- [Example of provisioning an Azure App Service](https://octopus.com/docs/runbooks/runbook-examples/azure/provision-app-service)
- [Example with AWS CloudFormation](TBD)
- [Example using Terraform](TBD)


## Conclusion
Implementing these types of self-service runbooks could reduce your time to market and give you an edge over the competition.