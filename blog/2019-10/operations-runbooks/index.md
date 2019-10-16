---
title: Introducing Runbooks for your Operations Team
description: Introducing Runbooks for your Operations team. It’s now possible to run operations and maintenance tasks like file clean-ups, backup and restore jobs, as well as disaster recovery failovers.
author: jessica.ross@octopus.com
visibility: public
bannerImage: operations-runbooks.png
metaImage: operations-runbooks.png
published: 2020-10-16
tags:
 - Product
---

![Illustration showing books running (i.e., Runbooks) through a server room](operations-runbooks.png)

Deployments are only part of what goes into making applications available to users. There are many other common tasks that need to be performed to keep applications operating smoothly. A large part of DevOps is development and operations teams working together, and Octopus is the perfect tool to use, given it already knows about your infrastructure, accounts, and project configuration.

If we take the Octopus website, Octopus.com, as an example, we use Octopus Deploy to deploy our website and manage our infrastructure, variables, certificate, and accounts. But we also have some routine and emergency tasks we do as part of *operating* the website. For instance, backing up the database, restoring it, testing the restore, removing PII from the database, and restoring the sanitized database to a test environment as well as failing over to a disaster recovery site.

Currently, these tasks are in a project’s deployment process, or they’re executed as separate scripts. Performing all these steps for a deployment doesn’t make sense, and doing a deployment just to backup a database doesn’t make sense either.

Operation Runbooks allows us to simplify the deployment process and enable the operational tasks to be run at different intervals to the deployment. Our list of Runbooks is:

1. Failover to the disaster recovery site.
2. Switch back to the primary production site.
3. Back up the database (and test the restore worked).
4. Refresh the test database with (sanitized) production data.

With Runbooks, the overview and task lists show us the exact operations and their runs performed separately, giving us a true picture of the state of our website.

## Operation Runbooks in Octopus Deploy

Operation Runbooks can be accessed from within a project. This means you can keep everything related to running an application together. If you have operations that apply to your infrastructure only and are not necessarily related to an application, like cleaning up files on machines, we suggest you create a separate project for these types of operations.

In the project menu, everything related to deployments is under a new **Deployments** menu-item. Runbooks are under the **Operations** menu, and you can expect to see this section grow as we add new Operations in the future. Resources that can be shared between Deployments and Operations are outside of these areas.

![Screenshot showing the new menu structure within a project](deployments-01.png)

## Creating and running a Runbook

Adding a Runbook can be done from the {{Operations>Runbooks}} section, and adding steps to a Runbook works the same as adding steps to a Deployment Process.

If you have operational steps within a Deployment Process, these can be cloned into a Runbook.

![Screenshot of the Runbooks screen](runbooks-01.png)

When Runbooks run, a snapshot is created at the time of the run, making it quicker to perform operations. We didn’t design Runbooks to rely on lifecycles, so you can run a Runbook on any environment as long as you have the permission.

![Animated gif of a Runbook being run](running-runbook.gif)

## Cloning steps

If you have steps in your deployment process that are more suited to a Runbook, you can clone the step to your Runbook.

## New permissions

You can enable or restrict users from running Runbooks with the new permissions `RunbookView` and `RunbookEdit` (for creating, editing, and deleting). Runbooks re-uses the existing `ProcessEdit`, `Deployment`, and `Release` permissions for runs and snapshots.

## Operation Runbooks early access

We’ve shipped RunBooks as an early access feature in [Octopus 2019.10](/blog/2019-10/octopus-release-2019.10/index.md), and we encourage you to try Runbooks for any processes that aren’t application specific.

Our goal with the early access program is to get feedback and validate its design. There are some limitations with Variables, Triggers, and other areas within the EAP, but the [Runbooks documentation](https://octopus.com/docs/deployment-process/operations-runbooks#current-limitations) provides all the details you need to get started as well as the current limitations.

We’d love feedback, so join the discussion on our [community slack](https://octopus.com/slack) in the `#runbooks` channel. You can also register for updates on our [public roadmap](https://octopus.com/company/roadmap) page.

## Conclusion

We’re excited to share this first release of Operation Runbooks and to see how teams use it in their projects.
