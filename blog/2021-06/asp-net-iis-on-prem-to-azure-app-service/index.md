---
title: Migrate an ASP.NET web app from IIS on-prem to an Azure App Service
description: How to migrate an existing ASP.NET web app from IIS on-premesis to an Azure App Service in the Cloud.
author: rob.pearson@octopus.com
visibility: public
published: 2022-06-02
metaImage: socialimage-github-actions-integration_2021.png
bannerImage: socialimage-github-actions-integration_2021.png
tags:
 - DevOps
 - Migration
---

![Migrate an ASP.NET web app from IIS on-prem to an Azure App Service](socialimage-github-actions-integration_2021.png)

Team are migrating their applications and infrastructure to the cloud. The natural migration path for ASP.NET web apps or services running on Microsoft's IIS web server on-prem is to App Services on Microsoft's Azure cloud platform. 

It's relatively straightforward to change the way your application is hosted but it's also important to ensure your CI/CD process is still repeatable and reliable. In this blog post, I'll walk through how to update the deployment process of an existing web application with Octopus Deploy shifting it from on-prem to the cloud. 

## Example application

We're using the Random Quotes web application as the example in this migration guide. This is a web application which retrieves and displays famous quotes randomly. It has an ASP.NET front end with a SQL Server backend. This is very simple however it helps illustrate the changes required. 

## Updating our deployment process

Our Octopus deployment process has two steps. In the on-prem version, it has a script step that executes a command line tool to update our database and deploy to IIS 

## Before

## After

## Migrating to Octopus Cloud (optional)

One commmon pattern we've see with our customers as they shift away from virtual machines on-premises is that they also look to shift their CI/CD platform from on-prem tooling to the cloud. For example: companies shift from using TeamCity and Octopus on-prem to TeamCity Cloud and Octopus Cloud. 

I'll briefly outline how to accomplish this with Octopus Deploy. We recently launched Project export and import support to make it easy to move projects from one Octopus instance to another. This is self-service meaning that individual developers can do this themselves without the need for operations teams to help. 





## Update your configuration variables

## Update your deployment process

## Blue-green deployments with runbook automation 

## Conclusion

Reword intro.