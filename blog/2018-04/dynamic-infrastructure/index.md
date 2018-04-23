---
title: "Managing Dynamic Targets"
description: Walkthrough of managing a QA environment in Azure
author: ben.pearce@octopus.com
visibility: private
published: 2018-04-20
tags:
 - Walkthrough
 - Azure
---

In 2018.5, we are introducing the ability to manage your Azure deployment targets from within your deployment process. 
Using the Azure Powershell modules you can create Resource Groups and Web Apps within your Azure subscription, but you could deploy your applications to them without some heavy lifting within Octopus.

In this blog post, I will walkthrough a basic example of how you could manage a Web Application in QA environment, hosted in Azure, from setup to tear down.

## Setup

First we need to configure Octopus to manage our new project.

### Create an Environment and configure dynamic infrastructure

By default, an environment is not allowed to have dynamic targets create or removed, so you will need to turn this on.

![Environment configuration](dynamic-infrastucture-environment-setting.png "width=500")

### Create a new lifecycle

So as to simplify our QA environment, and prevent it from deploying to other environments (such as Production), we can create a new [Lifecycle](https://octopus.com/docs/infrastructure/lifecycles) that only allows deployments to our new environment.

![QA Only Lifecycle](qa-only-lifecycle.png)

### Create tenants
    
In this example, I am using tenants to demonstrate how you can structure a QA environment. A tenant might represent a Tester, or a Customer

Create a script module 

Create a variable set
    - prefix and account variable

Create a project
    - configure tenants setting
    - configure variable set
    - configure deployment targets setting
    - configure process to use new lifecycle




## Deploy


## Tear down

Create a project for Tear down
    - configure variable set
    - add step
    - configure scheduled deployment



Setup a process with a powershell script