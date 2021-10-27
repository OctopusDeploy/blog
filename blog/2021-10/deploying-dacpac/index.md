---
title: Deploy SQL Server DACPAC with Octopus Deploy.
description: Learn how to deploy SQL Server DACPAC using Octopus Depoloy.
author: shawn.sesna@octopus.com
visibility: priovate
published: 2022-10-25-1400
metaImage: 
bannerImage: 
bannerImageAlt: 
isFeatured: false
tags:
 - 
---

Database Administrators (DBAs) will sometimes cringe at the mere mention of automating a database deployment.  Their job is to make sure that the server and databases remain available and healthy so any process outside of their control makes them nervous.  Introducing something that can make wholesale changes to the database structure or data automatically seems like a stark contrast to their duties.  However, using DACPAC with Octopus Deploy to automate deployment to SQL Server can assist you in your DevOps journey.  This post demonstrates automating database updates to Microsoft SQL Server using DACPAC and Octopus Deploy from build to deployment.

## Sakila
This post will deploy the [sakila](https://bitbucket.org/octopussamples/sakila/src/master/src/dacpac/mssql/) database to a Microsoft SQL Server.  This project contains tables, constraints, stored procedures, views, and user defined functions as to demonstrate the full capabilities of the Microsoft DACPAC technology.

::info
The sakila git repo contains source code for deploying the sakila database to multiple database technologies using different deployment methods.  This post will focus specifically on the Microsoft DACPAC version.
::

## Creating the build
