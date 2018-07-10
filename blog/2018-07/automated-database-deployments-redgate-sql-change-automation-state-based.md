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
- SQL Server Management Studio
    - Download for free [here](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms)

## Initial Setup

Most of the setup is pretty straight-forward.  With Redgate's SQL Toolbelt you will be prompted to install quite a bit.  Here is what you will need to install:

### Developer Machine
This is the machine where we will be making the schema changes and checking them into source control.  You will need to install the following:

- SQL Source Control
- SQL Prompt (it isn't required, but it makes your life so much easier)
- SSMS Integration Pack

### Build Server Agent

You will have to install a couple of plug-ins to get it all to work.  

- Jenkins
- TeamCity
    - Octopus - download [here](https://download.octopusdeploy.com/octopus-teamcity/4.34.0/Octopus.TeamCity.zip)
    - RedGate - download here

### Deployment Target
It is not recommended to install an Octopus Tentacle directly on SQL Server.  The [documentation](https://octopus.com/docs/deployment-examples/sql-server-databases#SQLServerdatabases-Tentacles) goes into further details why.  Instead we will be install the tentacle on a jumpbox which sits between Octopus Deploy and SQL Server.  For this you will need to install

- SQL Change Automation PowerShell 3.0
- SQL Change Automation

## Checking In To Source Control

## Build Tool Configuration

### TeamCity

### VSTS

### Jenkins

## Octopus Deploy Configuration

## Octopus Deploy Deployment

## Conclusion