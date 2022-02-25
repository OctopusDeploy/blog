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

The application scenario used as an example is legacy, because it is more likely to find a tricky legacy manual deployment that needs this technique and new applications you write today are more likely to have deployability baked into their design.

## Create a checklist

To demonstrate this technique, we'll make up a legacy system. You may already have a document or checklist that describes how to deploy the website, so we'll start with that.

You run a web application on multiple IIS servers, behing a load balancer. You deploy by taking each server out of balance in turn and applying the new version of the application.

The process for a each server is:

- Create a folder in d:\www\ with the version number of the app, i.e. d:\www\6.4.1
- Copy the built application into the new folder
- Open the previous version folder and copy the db.config and settings.config files into the new folder
- Stop the app pool
- Go to IIS Manager -> Sites -> AppSite and edit the Basic Settings
- Change the Physical Path to the new version folder and clock OK
- Start the app pool

This checklist will be our starting point for migrating the deployment into Octopus Deploy and then automating it.

TEMP SCRIPT

```
$server = hostname
$site = "UpDn"
$appPool = "UpDn"
$attempts = 0;
$connections = 0;

Stop-WebAppPool -Name $appPool

Do {
     Start-Sleep -Seconds 10
     
     $data = Get-Counter -ComputerName $server "\web service($site)\current connections"
     $connections = $data.CounterSamples[0].CookedValue
     $attempts++

     Write-Host $attempts, $connections
    
} while($connections -gt 0 -And $attempts -lt 20)

Write-Host 'Flushed'

Start-WebAppPool -Name $appPool
```


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

