---
title: Operations Runbooks Examples
description: Runbook automation examples in Octopus Deploy.
author: rob.pearson@octopus.com
visibility: public
bannerImage: blogimage-runbookslaunch.png
metaImage: blogimage-runbookslaunch.png
published: 2020-01-29
tags:
- Product
---

// TODO: Replace with release tour based on John's feedback. 5 mins of a example.
![Operations runbooks examples](blogimage-runbookslaunch.png)

We recently shipped Operations Runbooks for [Octopus Cloud](https://octopus.com/cloud) and [self-hosted](https://octopus.com/downloads) customers running Octopus 2019.11.0 or newer. Runbooks are the Ops in DevOps and automate routine maintenance and emergency operations tasks like the following.

- Infrastructure provisioning
- Database management
- Website failover and restoration
- Server maintenance

In this blog post, we look at what a runbooks is, why Octopus is useful for runbook automation and highlight some scenarios where runbooks offer much value.

<h2>In this post </h2>

!toc

## What is a runbook? 

Traditionally, runbooks document IT processes that keep your applications running smoothly, and most teams have something like this. They're often in the form of word docs, wiki pages or service management systems. It's common for people to print them out and tick off the steps as they walk through them.

Teams use runbooks for two core reasons:

1. Routine operations tasks, like database administration and service maintenance. 
2. Emergencies and incidents like website failovers and unplanned infrastructure outages. 

Runbook automation is a way to improve on traditional runbooks documentation. Automating the steps to execute operations procedures and resolve emergencies. This combination of runbook docs and runbook automation brings many benefits.

* Runbooks capture and share knowledge across teams and they are well suited to teams in a DevOps world. Developers, operations folks as well as on-call staff. Experts not required.
* Runbook automation is fast and reduces human error. Traditionally, runbooks are written processes, and while docs are good, automation is better. Scripts are testable, repeatable, and they can be improved over time.
* Reduced friction and incident resolution time. In emergency scenarios, runbook automation reduces the friction to resolving problems in a fast and efficient manner.

## Why use Octopus for Runbook automation? 

It's already possible to document and script operations processes, so why use Octopus Operations Runbooks?

* **Runbooks and deployment processes sit side-by-side.** Runbooks are designed to automate operations tasks, and they can share configuration settings, secrets, step templates, scripts, and more. Runbooks are lightweight automated processes that are executed against your infrastructure without going through a deployment lifecycle (i.e. dev, test, production).
* **Octopus is already aware of your infrastructure.** Runbooks leverage the infrastructure that your applications are deployed to, so there's nothing new to configure, reducing the friction to getting started with running operations processes.
* **Security, permissions, and auditing.** Runbooks are managed and executed by Octopus, so there’s a complete audit trail that can be reviewed in retrospectives, making it easy to see what happened, when and why, and if anything needs to be changed. Octopus enables teams to control who can execute which Runbooks in what environments with advanced security and permissions.
* **Better emergency management and reduced incident resolution time.** With Runbooks in Octopus, no local tooling is required, no additional permissions are needed, and you have a detailed audit log of everything. On-call team members can quickly execute Runbooks without worrying about dependencies or infrastructure.
* **Discoverability and visibility.** Octopus creates a central location for teams to manage, control, audit, schedule, and run runbooks. You can see when a runbook was last ran, you can see the changes to the runbook, and you can run the same runbook against different environments. Team members can easily find a runbook, and click a big green button to run it. And everyone can see the output from the last run and whether it succeeded or not.
* **World-class scheduling and execution.** Execute runbooks on-demand or schedule them at any frequency.

## Runbook scenarios

I want to highlight some scenarios where runbooks shine. 

### Web application and web service recovery

// TODO: Add Video w/ a runbook screenshot as the cover page.

It's common for web applications and services to have problems due to a vast range of problems. It could be due to problems like memory leaks or unexplained performance issues. In the Windows Server world, the problems are commonly resolved by recycling the IIS web server app pool, and in the Linux world, it's common to reload NGINX or starter a docker container hosting the server. While this doesn't address the underlying issue, but it enables the teams to resolve the problem so they can work on a proper fix. Sometimes, this happens so sporadically, executing a runbook is the long term solution. 

In Octopus, this is a very straightforward runbook with a single Script step using PowerShell, Bash or Python. 

### Database administration

// TODO: Add Video w/ a runbook screenshot as the cover page.

Database management is often a slow process that requires emailing your database administration team (DBA) to get things done. It is common for DBAs to distrust developers or operations folks. Databases are a critical part of modern applications, and there are a plethora of database tasks that can be requested, including: 

* Database backups
* Refreshing test databases with sanitised production data
* Rebuilding indexes to resolve slow queries
* Revoking permissions when an employee leaves the company

These processes are ideal for runbook automation. Building these runbooks in Octopus in collaboration with database administrators is a great way to build trust and reduce the number of DBA requests for routine maintenance.  

In Octopus, it's straightforward to configure runbooks for any of these tasks with the following steps.
- SQL step template or custom script
- Slack or email notification

## Conclusion

Operations Runbooks has shipped, and it helps teams keep their applications running smoothly. Runbooks in Octopus brings many benefits from its history of release management and deployment automation. They help teams automate operations tasks like routine maintenance and recover when things go wrong. 