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

You might be new to Octopus Deploy and need a strategy for taking manual deployments and automating them.

You might be an existing Octopus Deploy customer who still has a manual deployment process for a legacy app that you want to track.

## Create a checklist

Example manual process... we'll automate some or all of this later... including a step template that knocks out a couple of steps in one hit.


A software-as-a-service company that deploys the same code to 100 client sites, hosted on an IIS virtual machine, with separate config file changes for each client.

- Copy the build application files to the server, to the d:\release folder
- Delete the web.config file from d:\release as each client has their own customised config values
- Check that the web.config file has definitely been deleted
- Stop IIS
- Copy the files from d:\release into d:\wwwroot\{client} folders
- Start IIS


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

