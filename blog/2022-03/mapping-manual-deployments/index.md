---
title: Mapping manual deployments with Octopus Deploy
description: Find out how to map your manual deployments to create a template that helps you start your automation journey.
author: steve.fenton@octopus.com
visibility: private
published: 9999-01-01-0000
metaImage: 
bannerImage: 
bannerImageAlt: 
isFeatured: false
tags: 
  - DevOps
  - Getting Started
  - Product
---

While Octopus Deploy makes complex deployment automation easy, you might have a scenario where you are still running a manual deployment. We have talked to customers with a legacy application that needs some special steps during the release process, or who have esoteric technology that makes it hard to work out where to start the automation journey.

In this post, you'll find out how you can model your manual deployments with Octopus Deploy and the benefits this will bring to your release process.

If you are new to Octopus, you might find this a useful way to advance from your current deployment process to full automation. Existing customers may find that this technique allows them to bring to their manual releases all the benefits of approvals and tracking that they get with automated deployments.

## Creating a checklist

When you have a manual release process, there is usually a document or checklist that explains the steps required to install the software along with who can perform each step. Before you re-create this in Octopus, it is worth spending some time refining the stages to agree who does what, in which order.

If your document is lengthy, you might find you can divide it with headings that will give you a natural task list.

You should end up with something like the following checklist:

| Step   | Title                                                        | Who  |
|--------|--------------------------------------------------------------|------|
| 1      | Back-up the database                                         | DBA  |
| 2      | Upgrade the database                                         | DBA  |
| 3      | Copy the new application to a temporary folder on the server | Ops  |
| 4      | Delete the configuration file in the temporary folder        | Ops  |
| 5      | Copy the live configuration file into the temporary folder   | Ops  |
| 6      | Add any new settings to the configuration file               | Ops  |
| 7      | Copy the temporary folder into the live folder               | Ops  |
| 8      | Check the application loads as expected                      | Test |

If you didn't have a checklist before you started this exercise, you are likely to find that the checklist already increases the reliability of your deployments by ensuring the steps all occur and are done in the right order.




Notes from Ryan

https://github.com/OctopusDeploy/blog/blob/manual/blog/2021-09/modeling-manual-deployments/index.md

Old stuff, like VB6 DLLs - Esoteric
Manual intervention
Paste log files or output as comments to capture the action done
The approval steps become "check it worked" - and eventually as you gain confidence and remove the manual steps

At least track it - something happened.

Word doc / wiki page / etc... pages of conditional instructions - put them into Octopus and it can be the documentation with extra benefits.

If you are using manual steps it doesn't really add to you cost.

Customers not in control of target infrastructure - so the manual process lets them track the deployment being done in an ops group.

Completely manual customers - i.e. no source control, no builds, etc



## Create a checklist

To demonstrate this technique, we'll make up a legacy system. You may already have a document or checklist that describes how to deploy the website, so we'll start with that.

You run a web application on multiple IIS servers, behing a load balancer. You deploy by taking each server out of balance in turn and applying the new version of the application.

The deployment process is:



This checklist will be our starting point for migrating the deployment into Octopus Deploy to help run the manual deployment.

## Model your checklist in Octopus Deploy

You may find that your process deploys multiple components. If you can split the checklist into independent deployments, you should do so. This might not be possible if the process is highly coupled and you would need to revisit the process after you have thought about how you could isolate the deployments.

Map each checklist item as a manual intervention

## Perform a deployment

Complete your next deployment with Octopus Deploy

## Adjust

Adjust the process if it isn't right and repeat!

## Automate a small part

Pick a step to automate.

How to choose a step? You want to build confidence, so start with an easy step.

Each step that you automate will free up time that you can use to tackle more complex steps.

implement it - create an additional step (before or after?) your manual step

When you pick a step template, you might find it does more than you expected, for example a checklist that had separate steps for configuring a web server could be achieved in a single build-in step in Octopus Deploy. If this happens, disable all of the manual steps that have been achieved in the automation.

### Sources of help

- Docs
- Community Slack
- Where else?

## Be pragmatic

There might be constraints that mean automating a particular step is difficult at this current time. That's okay. Just keep a note of any manual steps and why you could automate them and revisit it later.

## Increase your deployment frequency

Once you have introduced more automation, you should review how often you deploy. If your deployment was only released once a month because it took so long to roll it out, you could look at moving to weekly releases.

Frequent deployments are correlated to high-performance, not simply because releasing often is a high-performance trait, but beause of what you learn and adjust to achieve it.

