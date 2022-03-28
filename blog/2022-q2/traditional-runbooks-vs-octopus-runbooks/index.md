---
title: Traditional runbooks versus Octopus Runbooks
description: As part of our series about Runbooks, we look at how Octopus Runbooks solves the problems with traditional runbooks.
author: andrew.corrigan@octopus.com
visibility: private
published: 2022-04-11-1400
metaImage: blogimage-placeholder.png
bannerImage: blogimage-placeholder.png
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - DevOps
  - Runbooks Series
  - Runbooks
---

We talk a lot about how Octopus can complete your Continuous Integration and Continuous Deployment (CI/CD) pipeline, but did you know Octopus's Runbooks feature can help with your operations tasks too?

In this post we explore the benefits of Octopus Runbooks as part of DevOps, the problems they solve, plus how to set them up.

## What is a runbook?

In the simplest terms, a runbook is a step-by-step guide to complete a routine or emergency computer process.

In the old days of IT operations, teams would collate these guides into physical books or folders, pulled from the shelf only when needed. Nowadays, they're commonly found in documents on shared network drives, wikis, or knowledge bases.

Say, for example, an application server has a known issue caused by a Windows service that hangs occasionally. Support teams may have a runbook (or knowledge article) to guide support members through restarting that service. Other typical runbooks could include:

- Database backups and restores
- Server maintenance such as upgrades, patching, or file tidy-up
- Server, service or web application restarts
- Troubleshooting steps with decision trees

Whether physical or otherwise, manual runbooks have some problems, as they can be:

- Time consuming - manually working through guides and branching steps takes time, more so for those unfamiliar with a system
- Prone to human error - Mistakes can happen to the best of us, even with the simplest instructions. After all, pobody's nerfect!
- Outdated - internal documentation upkeep often falls down the priority order in busy teams. This means runbooks are often out-of-date or lack consistency.
- Annoying to manage access for - your operations team likely need access to countless systems. Risk naturally increases alongside the number of people with access to something, or the more access one person has. But also, you don't want support members to find they don't have access when they really need it.

These are all problems you can avoid if you're an Octopus user... 

## The benefits of Octopus Runbooks

Let's look at exactly how Octopus helps solve the problems with traditional runbooks, plus some of the other benefits.

### Runbook automation

As a product, we developed Octopus on the belief that repeatable deployments means robust deployments. Octopus Runbooks follows that same belief.

This reduces the risk of human error and speeds up processes. Just set your steps once and trigger a runbook with one click.

### Octopus Runbooks already understand your environments

If you're a customer, your Octopus instance already connects to the infrastructure you deploy to. Any runbooks you create in Octopus also uses those integrations, so there's nothing new to set up for operations tasks.

Octopus Runbooks aren't tied to your environment's deployment lifecycles either. You can run them against any environment or deployment target whenever you need.

### Octopus Runbooks helps with security and saves network admin

In our earlier example, where support needs to restart a service on a Windows server, you'd need to give them access to the server via one of the following:

- Permission groups
- Shared network accounts
- Local admin rights

If that same support team looks after many systems (as is the case in operations), figuring out exactly what they have or need access to can get confusing.

Given Octopus already connects to your deployment targets, our runbooks mean you don't need to assign direct access to infrastructure for operations tasks. So, if on-call support needs to restart something, they only need access to Octopus and that runbook. Octopus manages the rest.

### Octopus has detailed audit trails

Octopus also provides full audit logs of every triggered runbook, so you can always see:

- Who did what, when and why
- Runbook success and failure - know when and why it's time to update your runbooks
- Information needed for incident response reports

### If you can create a deployment process, you can create a runbook

The processes for creating deployments and runbooks are very similar. With both, you set steps using a combo of predefined actions or whatever scripting language you're comfortable with.

In fact, we'll show you. Let's walk through the creation process with a simple runbook.

## Creating a simple runbook in Octopus

This is a short example on how to create a basic runbook that won't affect any of your projects or environments. You can follow along if you're an existing user or by [signing up for a free trial](https://octopus.com/start).If you don't want to follow along but would still like to see the end result, we set up an [example instance with this runbook](https://tenpillars.octopus.app/app#/Spaces-82/projects/starter-runbooks/operations/runbooks/Runbooks-181/process/RunbookProcess-Runbooks-181) that's accessible to guests.

This guide assumes you already:

- Set up some environments and projects in Octopus. If you haven't, check out our [guide to your first deployment](https://octopus.com/docs/getting-started/first-deployment).
- Have Slack, a channel you can post to, and have created a Webhook. See [Slack's Webhook documentation](https://api.slack.com/messaging/webhooks) for more information.

This runbook will:

- Run a 'Hello World' script
- Notify a Slack channel when the runbook completes successfully

1. Open and log into Octopus.
1. Click **Projects** and select an existing project from the list.
1. Click **Operations** from the left-hand menu.
1. Click **GO TO RUNBOOKS**.
1. Click **ADD RUNBOOK**.
1. Give your new runbook a suitable name and description and click **SAVE**.
1. Click **DEFINE YOUR RUNBOOK PROCESS**.
1. Click **ADD STEP**.
1. Select **Script**, hover over **Run a Script** from the results and click **ADD**.
1. Change the following settings, leave everything else as default and click **SAVE**:
   - **Step Name** - give the step a descriptive name
   - **Execution Location** - select **Run on the Octopus Server** or **Run once on a worker** depending on your Octopus setup.
   - **Inline Source Code** - select the **PowerShell** radio button and enter the following into the code box: `Write-Host 'Hello, World!'`
1. Now we can add the step that sends a message to Slack when the runbook completes successfully. Click **ADD STEP** again. 
1. Search for `slack`, hover over **Slack - Send Simple Notification** from the results and click **ADD**. Click **Save** if prompted to save the step to your instance's templates.
1. Complete the following settings, leave everything else as default and click **SAVE**:
   - **Step Name** - give the step a descriptive name.
   - **Execution Location** - select **Run on the Octopus Server** or **Run once on a worker** depending on your Octopus setup.
   - **Hook URL** - copy in your Slack Webhook URL. If others can see your Octopus instance or you're creating a real runbook, consider adding your webhook as a [project variable](https://octopus.com/docs/projects/variables). You can then securely call the webhook with the following syntax `#{variable-name}`.
   - **Channel handle** - enter the exact Slack channel you'd like to post the message to.
   - **Message** - type in the success message you'd like Octopus to send to Slack.
 1. Click **RUN...** to test the runbook. Depending on your Octopus setup, you may need to select environments. If so, select any environment (it won't matter which as this runbook won't change anything) and click **RUN** again.
 1. Wait for the runbook to finish and check Slack for the message we set earlier.

When creating a runbook for real, you must click **PUBLISH** to make it available to other team members and Octopus trigger events.

## What's next?

This is just a taster to show you the benefits of Octopus Runbooks and how easy it is to set them up.

In upcoming series posts we'll walk you through some real use-cases, including how to use Octopus Runbooks to:

- Respond to vulnerabilities
- Create standardized support emails
- Run 'smoke tests' on your infrastructure
- Manage cloud costs with instance scheduling
- Manage 'shadow IT' resources
- Calculate DORA metrics
- Set up Linux servers with Bash scripting

In the meantime, check our [Octopus Runbook documentation](https://octopus.com/docs/runbooks/runbook-examples) for even more examples.

Happy deployments!