---
title: Operations Runbooks in Octopus
description: Runbook automation examples in Octopus Deploy.
author: rob.pearson@octopus.com
visibility: private
bannerImage: blogimage-runbookslaunch.png
bannerImageAlt: Runbooks examples for operations tasks
metaImage: blogimage-runbookslaunch.png
published: 2021-01-01
tags:
- DevOps
- Runbooks
---

![Runbooks examples for operations tasks](blogimage-runbookslaunch.png)

The focus of this post is all about examples in an SEO focused way. We want to be own runbook search terms.

---

We recently shipped Operations Runbooks for [Octopus Cloud](https://octopus.com/cloud) and [self-hosted](https://octopus.com/downloads) customers running Octopus 2019.11.0 or newer. Runbooks are the Ops in DevOps and automate routine maintenance and emergency operations tasks like the following.

- Infrastructure provisioning
- Database management
- Website failover and restoration
- Server maintenance

In this blog post, we're going take a look at runbooks, why they're valuable and several examples.

<h2>In this post </h2>

!toc

## What is a runbook?

Traditionally, runbooks document IT processes that keep your applications running smoothly and most teams have something like this. They're often in the form of word docs, wiki pages or service management systems. It's common for people to print them out and tick off the steps as they walk through them.

Teams refer to runbooks for two main reasons:

1. Routine operations tasks like database administration and sevice maintenance.
2. Emergencies and incidents like website failovers and unplanned infrastructure outages.

Runbook automation is a way to improve on traditional runbooks documentation. Automating the steps to execute operations procedures and resolve emergencies.

Runbooks and runbook automation brings a number of benefits.
* Runbooks capture and share knowledge across teams and they are well suited to teams in a DevOps world. Developers, operations folks as well as on-call staff. Experts not required.
* Runbook automation is fast and reduces human error. Runbooks are traditionally documented processes and while docs are good, automation is better. Scripts are testable, repeatable and they can be improved over time.
* Reduced friction and incident resolution time. In emergency scenarios, runbook automation reduce the friction to resolving problems in a fast and efficient manner.

## Why use Octopus for Runbook automation?

It's already possible to document and script operations processes, so why use Octopus Operations Runbooks?

* **Runbooks and deployment processes sit side-by-side.** Runbooks are designed to automate operations tasks, and they can share configuration settings, secrets, step templates, scripts, and more. Runbooks are lightweight automated processes that are executed against your infrastructure without going through an deployment lifecycle (i.e. dev, test, production).
* **Octopus is already aware of your infrastructure.** Runbooks leverage the infrastructure that your applications are deployed to, so there's nothing new to configure, reducing the friction to getting started with running operations processes.
* **Security, permissions, and auditing.** Runbooks are managed and executed by Octopus, so there’s a complete audit trail that can be reviewed in retrospectives, making it easy to see what happened, when and why, and if anything needs to be changed. Octopus enables teams to control who can execute which Runbooks in what environments with advanced security and permissions.
* **Better emergency management and reduced incident resolution time.** With Runbooks in Octopus, no local tooling is required, no additional permissions are needed and you have a detailed audit log of everything. On-call team members can quickly execute Runbooks without worrying about dependencies or infrastructure.
* **Discoverability and visibility.** Octopus creates a central location for teams to manage, control, audit, schedule, and run runbooks. You can see when a runbook was last ran, you can see the changes to the runbook, and you can run the same runbook against different environments. Team members can easily find a runbook, and click a big green button to run it. And everyone can see the output from the last run and whether it succeeded or not.
* **World-class scheduling and execution.** Execute runbooks on demand or schedule them at any frequency.

## Runbook examples

I'd like to highlight some runbook examples and I'll add new ones over time. Add a comment with your favourite runbook or automated operations process.

### Routine maintenance

#### Recycle IIS App Pool

TODO: Screenshot

Most developers who work on the Microsoft stack have experience sporadic web application or Windows service problems like memory leaks or unexplained performance issues. The problems are commonly resolve by recycling the IIS web server app pool or restarting a windows service. While this doesn't address the underlying issue, it does quickly resolve problems and enable customers to continue to use the applications they support.

In Octopus, this is a very straightforward runbook with a single Script step that executes the following PowerShell script.

```powershell
# Example 1: Restart IIS app pool
Restart-WebAppPool MyAppPool
```

#### Restart NGINX or restart a Docker container (containerized application)

If

 NGINX or re

Similar to the first runbook example, another common operations task is to restart an NGINX (via systemd) or restart an containerized application.

In Octopus, this is a simple runbook with a single script step that executes the following bash script.

```bash

# Restart NGINX
sudo systemctl reload nginx

```

#### Provision test infrastructure

TODO: Screenshot

Description

#### File clean-up

TODO: Screenshot

It's not uncommon for web services that do run various business processes to leave artifacts on file systems or cloud storage and this can exceed it's limits.

```bash

# TODO

```

### Emergency scenarios

#### Website failover

TODO: Screenshot

Most modern websites are highly available nowadays have have some sort of elastic or scaling capability. This is further enhanced with cloud platforms like [Microsoft Azure](https://azure.microsoft.com/) and [Amazon Web Services](https://aws.amazon.com/). That said, it's not uncommon for entire regions to have outages and wreak havoc with websites, databases and other infrastructure.

This is where a disaster recovery site can help in a different cloud or region however it can be complicated to make the switch over. This is often captured in runbook documentation however it's the ideal candidate to be automated. The example above shows

#### Running a database administration scripts

Sudden increases in activity for a website or service can greatly change the performance of a database and affect the service. In situations like this, you could either

```sql

-- TODO

```

NOTE: This type of runbook could be considered a

#### Server running out of disk space

TODO: Screenshot

Screenshot

Description

## Conclusion

Operations Runbooks has shipped and it helps temas keep their applications running smoothly. Runbooks in Octopus bring a lot of benefits from its history of release management and deployment automation. They help teams automate operations tasks like routein maitenance and recover when things go wrong.
