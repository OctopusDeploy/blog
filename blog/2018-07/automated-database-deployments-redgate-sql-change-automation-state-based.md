---
title: Automated Database Deployments using State Based Redgate SQL Change Automation
description: Automated Database Deployments using State Based Redgate SQL Change Automation
author: bob.walker@octopus.com
visibility: private
published: 2018-07-16
tags:
 - Database Deployments
---

## Introduction

Previous blog posts discussed why [you need automated database deployments](https://octopus.com/blog/automated-database-deployments-series-kick-off) and [tips on getting started](https://octopus.com/blog/automated-database-deployments-iteration-zero) down that path.  Enough talk, it is time for action!  This article will walk you through setting up an automated database deployment pipeline using the [state based approach](https://www.red-gate.com/products/sql-development/sql-change-automation/approaches) for [Redgate's SQL Change Automation](https://www.red-gate.com/products/sql-development/sql-change-automation/).  I picked this tool to start with because it is easy to setup, integrates nicely with SSMS, and...well...I already had a demo setup.  I'm also a [little biased](https://www.red-gate.com/hub/events/friends-of-rg/friend/BobWalker) towards Redgate's tooling.  So there's that.

The end goal of this article is for you to have a working proof of concept for you to demo to your team or leaders within your organization.

## Prep Work

For this demo you will need a SQL Server instance running, an Octopus Deploy instance and a CI server.  I recommend using your local machine for this but it is up to you.  

## Tools Needed

If you would like to follow along at home you will need the following:

- Octopus Deploy
    - Get 45-day free trial for on-premise [here](https://octopus.com/licenses/trial).
    - Get 30-day free trial for Octopus Cloud [here](https://octopus.com/account/register).
- Redgate SQL Toolbelt
    - Get 14-day free trial [here](https://www.red-gate.com/dynamic/products/sql-development/sql-toolbelt/download).  
- CI Tool (pick one)
    - Jenkins - download [here](https://jenkins.io/download).
    - TeamCity - download [here](https://www.jetbrains.com/teamcity/download/).
    - TFS - download [here](https://visualstudio.microsoft.com/tfs/).
    - VSTS - try [here](https://visualstudio.microsoft.com/team-services/).
    - Bamboo - download [here](https://www.atlassian.com/software/bamboo/download)
- SQL Server Management Studio (SSMS)
    - Download for free [here](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms)
- SQL Server
    - SQL Express - download [here](https://www.microsoft.com/en-us/sql-server/sql-server-editions-express)
    - SQL Developer - download [here](https://www.microsoft.com/en-us/sql-server/sql-server-downloads)

## Installing Software

Most of the setup is pretty straight-forward.  Follow the wizards and the prompts and what-not.  

### Developer Machine
This is the machine where we will be making the schema changes and checking them into source control.  Redgate's SQL Toolbelt will prompt to install quite a bit.  

You only need to install the following:
- SQL Source Control
- SQL Prompt (it isn't required, but it makes your life so much easier)
- SSMS Integration Pack

### Build Server

Both Octopus Deploy and Redgate have plug-ins support for the major build servers.  For ease of use I have included the download links below.  

- Jenkins
    - Octopus - download [here](https://download.octopusdeploy.com/octopus-tools/4.37.0/OctopusTools.4.37.0.zip) - Please note, you can have Jenkins interact with Octopus by using octo.exe.  You can read more about that [here](https://octopus.com/docs/api-and-integration/jenkins)
    - Redgate - download [here](https://plugins.jenkins.io/redgate-sql-ci)
- TeamCity
    - Octopus - download [here](https://download.octopusdeploy.com/octopus-teamcity/Octopus.TeamCity.zip)
    - RedGate - download [here](https://www.red-gate.com/dlmas/TeamCity-download)
- VSTS/TFS
    - Octopus - download [here](https://marketplace.visualstudio.com/items?itemName=octopusdeploy.octopus-deploy-build-release-tasks)
    - Redgate - download [here](https://marketplace.visualstudio.com/items?itemName=redgatesoftware.redgateDlmAutomationBuild)
- Bamboo
    - Octopus - download [here](https://marketplace.atlassian.com/apps/1217235/octopus-deploy-bamboo-add-on?hosting=server&tab=overview)
    - Redgate - download [here](https://marketplace.atlassian.com/apps/1213347/redgate-dlm-automation-for-bamboo?hosting=server&tab=overview)

### Deployment Target
It is not recommended to install an Octopus Tentacle directly on SQL Server.  The [documentation](https://octopus.com/docs/deployment-examples/sql-server-databases#SQLServerdatabases-Tentacles) goes into further details why.  Instead we will be install the tentacle on a jumpbox which sits between Octopus Deploy and SQL Server.  For security you have two options, you can use integrated security, which will require you to set the tentacle to run as an service account or user account.  Here is [some documentation](https://octopus.com/docs/infrastructure/windows-targets/running-tentacle-under-a-specific-user-account) on how to configure that.  

For the jumpbox you will need to install the following items:
- SQL Change Automation PowerShell 3.0
- SQL Change Automation

## Sample Project

For this walk-through I modified the RandomQuotes project used in previous Will It Deploy videos.  If you haven't had the chance to check them out you are missing out.  Do yourself a favor and watch them.  Each episode is less than around 15 minutes.  You can find the playlist [here](https://www.youtube.com/playlist?list=PLAGskdGvlaw13QRF-ypT9h83QTPutlbMn).  

The source code for this sample can be found here.

## Checking In To Source Control

Redgate's SQL Change Automation state based approach saves everything in source control as create scripts.  This is done via SSMS using SQL Source Control and the SSMS plugin.  

## Build Tool Configuration



## Octopus Deploy Configuration

## Octopus Deploy Deployment

## Conclusion