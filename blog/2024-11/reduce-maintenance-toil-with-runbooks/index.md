---
title: Free your teams from maintenance tasks with runbooks
description: Find out how to reduce toil by converting manual operations tasks to automated runbooks.
author: steve.fenton@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: blogimage-runbooks-2022.png
bannerImage: blogimage-runbooks-2022.png
bannerImageAlt: Split view of a pile of folders and a laptop screen with an Octopus arm emerging out the side.
isFeatured: false
tags: 
  - DevOps
  - Runbooks
---

Octopus added a feature to help you automate your operations runbooks in 2019. In our recent webinar, Michael Richardson and I discussed when runbooks are available, why automating them with Octopus is beneficial, and how people use them in the real world.

You can [watch the webinar](https://www.youtube.com/live/UEysbmos2T0?t=267s) or read on to find a summary of all the key points.

## What's a runbook?

A runbook is a set of steps that you need to follow to complete a task. Originally, this would have been something a senior team member understood, and they would pass it down to new people like our ancestors passed on stories.

Runbooks cover routine and emergency operations tasks, such as generating database backups or switching the firewall into strict mode when a web application is under attack.

The success of early runbooks depended greatly on the accuracy of the knowledge transfer and the memory of the people learning how things worked. Each team member would know a subset of the runbooks in use as they usually picked them up as needs presented themselves.

Eventually, people realized it would be a good idea to write down these runbooks. They were often stored in a large lever-arch file in the ops team space so anyone could grab it when they needed to perform a task. When organization wikis replaced paper files, runbooks migrated into wiki pages.

To make runbooks easier, people would create script snippets to perform parts of the process. You'd have a wiki page describing all the steps, but some steps would have an associated script that would take care of a standard part of the process.

In all these cases, runbooks were high risk. It was easy to perform steps in the wrong order or to miss a step entirely. It was also common for an unintended side effect to be introduced by missing an essential flag on a command, through a copy/paste error, or by executing a step against the wrong infrastructure or environment.

![The four ages of runbooks](four-ages-of-runbooks.png "width=500")

To remove the risk from runbooks, a more futuristic form of automation was needed that would ensure runbooks were secure and reliable. You needed to be able to trigger a runbook with the push of a button or automatically based on a schedule.

Your confidence never increases if you need manual steps as part of a runbook. There's always a chance you'll miss a step or make a mistake. When a runbook is fully automated, as in Octopus, your confidence increases each time you run it.

## The origins of Octopus runbooks

We created [Octopus Runbooks](https://octopus.com/docs/runbooks) because we could see customers using our deployment features to run tasks such as database backups. Customers would have a project group containing software deployments and associated utility tasks.

We could see opportunities to improve the experience around these tasks by making them part of a deployment project, reducing clutter on the dashboard, and letting them access the project's variables. Instead of creating a release to trigger a new run, you could push a button to start a runbook.

First-class support for runbooks arrived in 2019, and within 5 years, it had become one of our most popular features.

## The benefits of runbooks

When manual steps are needed for a traditional runbook, someone must access the infrastructure, whether a virtual machine, a cloud portal, or a command-line tool like `kubectl`. This isn't access you provide to everyone.

Access is only usually given to your most senior people, so these routine toil tasks fall to them. These low-value repetitive tasks take your most experienced team members away from valuable work.

This also impacts those without access, as they must raise their requests and wait for them to be done. This delays work elsewhere.

With fully automated runbooks, you can make them available as permission-based self-service actions that let people progress their tasks without raising tickets. They don't need to wait for someone to be available to do them manually, and your most experienced technical team members no longer need to action these requests.

For example, if a tester needs to clear a cache as part of their testing, they can be granted permission to perform this runbook specifically on the test environment. They can instantly unblock their testing, and the software version can progress more rapidly through the deployment pipeline.

With the runbook automated in Octopus, nobody needs elevated access rights to perform these routine tasks, and there is no chance of forgetting a command line flag or making an unintentional change while you're signed in.

## How people are using runbooks

We looked at 200,000 runbooks across 3,200 organizations to find inspiration for your runbook journey. The most common runbook categories are listed below:

1. Restart/shutdown
2. Export/import/report
3. Cleanup
4. Rotate keys/secrets
5. Provision infrastructure
6. Backup

![The common runbook categories](runbook-categories.png "width=500")

These 6 categories account for around half of all runbooks. The remaining half is a mixture of expected and surprising ideas.

Less common uses that may still be useful as inspiration for your situation:

- Clearing caches
- Updating log settings
- Resetting data for a test environment
- DNS
- Scale up/down or resize an instance

Less expected reasons to use runbooks:

- Apply printer settings
- Office automation, such as turning lights on and off or starting a coffee machine

There was also a runbook called "Happy Crisis Handler," which reminds us that runbooks can bring calm to stressful incident management situations.

## Reduce toil, increase joy

Runbooks can remove toil work for technical team members and reduce wait times for everyone else. Octopus makes runbooks as joyful as deployments with familiar tools optimized for utility tasks. We recently announced [config as code for runbooks](https://octopus.com/blog/introducing-config-as-code-runbooks), which lets you store your runbook process in version control and use branches and pull requests to manage them.

If you want to know more about runbooks, you can [watch the webinar](https://www.youtube.com/live/UEysbmos2T0?t=267s) that accompanies this article and check our [webinars page](https://octopus.com/webinars) to sign up for the "Introducing Config as Code for Runbooks" webinar.

Happy deployments (and runbooks)!