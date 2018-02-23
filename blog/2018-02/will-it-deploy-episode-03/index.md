---
title: Deploying to SQL Server with Entity Framework Core - Will it Deploy? Episode 3
description: We try to automate the deployment of a SQL Server Database using Entity Framework Core to manage our database structure and data.
author: rob.pearson@octopus.com
visibility: public
published: 2018-02-24
metaImage: metaimage-will-it-deploy.png
bannerImage: blogimage-will-it-deploy.png
tags:
 - Will it Deploy
---

Welcome to another **Will it Deploy?** Episode where we try to automate the deployment of different technologies with Octopus Deploy.  In this episode, we're trying to deploy a Microsoft SQL Server database using Entity Framework Core migrations to an Amazon Web Services (AWS) virtual machine (VM).

<iframe width="560" height="315" src="https://www.youtube.com/embed/0XfVDc71OpU" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

## Problem

### Tech Stack

Our app is a quote generator called [Random Quotes](https://github.com/OctopusSamples/WillItDeploy-Episode003). The application is pretty simple but it allows us to illustrate how to deploy database changes and updates.

* Microsoft [ASP.NET Core 2.0](https://docs.microsoft.com/en-us/aspnet/core/) web app
* [Entity Framework Core 2.0](https://docs.microsoft.com/en-us/ef/core/) framework
* Microsoft [SQL Server 2017](https://www.microsoft.com/en-au/sql-server/) database

Kudos to our marketing manager [Andrew](https://twitter.com/andrewmaherbne) who has been learning to code and built the first cut of this app. Great work! 

### Deployment Target

![Amazon web services logo](aws-logo.png "width=200")

* AWS - [EC2](https://aws.amazon.com/ec2) virtual machine 
* Microsoft [Windows Server 2016](https://www.microsoft.com/en-au/cloud-platform/windows-server)

## Solution

So will it deploy? **Yes it will!** Our deployment process looks like the following.

![Octopus deployment process](deployment-process.png "width=500")

Then we add the following steps to successfully deploy our database changes and web app.

- Octopus **Deploy a Package** step to copy our database scripts to our database deployment target
- Octopus Community Contributed step template -  **[SQL - Execute Script File](https://library.octopusdeploy.com/step-template/actiontemplate-sql-execute-script-file)** to execute our Entity Framework Core migration script agaist our SQL Server database. 
- Octopus **Deploy to IIS** step to deploy our ASP.NET Core web application

This project uses the following variables to store our app settings, database connection details and web app configuration.

![Project variables](project-variables.png "width=500")

This episode's [GitHub repo](https://github.com/OctopusSamples/WillItDeploy-Episode003) contains all the resources and links used in this video.

### Wrap-up

We hope you enjoyed this episode as we have many more in the works! If there's a framework or technology you'd like us to explore, let us know in the comments.

Don't forget to subscribe to our [YouTube](https://youtube.com/octopusdeploy) channel as we're adding new videos regularly. Happy deployments! :)
