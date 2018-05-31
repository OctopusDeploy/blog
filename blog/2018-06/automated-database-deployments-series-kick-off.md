---
title: "Automated Database Deployments Series Kick Off"
description: Automated Database Deployments Series Kick Off
author: bob.walker@octopus.com
visibility: public
published: 2018-06-04
tags:
 - Database Deployments
---

The most nerve racking part (for me) for any deployment is the database.  Deploying the code is far less stressful.  If something wasn't right the code could easily be rolled back.  And it is the same code which had been tested in dev, and QA, and pre-production.  There should be zero surprises with the code.  

Databases are not nearly as flexible.  Imagine there is an error in a script and the names of all the users gets deleted.  There isn't a good way to roll that back.  Sure a backup could be restored.  But when was that backup taken?  Have any users been in the system since that backup?  What data will be lost if the backup is restored?

Prior to working at Octopus Deploy I was a lead developer on a loan origination system for a large bank in the United States.  At any given point a billion US dollars was in flight inside that system.  Multiple people would work on a loan.  A financial officer might spend a half hour entering financial information about the customer.  While an underwriter could spend an hour researching the customer and placing any notes and conditions on the loan.  Needless to say, restoring a database backup was a last resort.  Telling the underwriter who spent 45 minutes entering in information that they have to do it all over again can happen only so many times before they would want to hunt me down.

Everything in the deployment was automated.  Except the databases.  The database change scripts were manually created by a database developer prior to the deployment.  It was common to see them spending a full business day putting the script together.  Chances are the first time the script was ran in production was the first time the script was ever ran.  Sure we used tools like Redgate's SQL Compare.  But that could only do so much.  

Due to the risk, namely of the data, deployments could only occur after hours, evenings or weekends.  This allowed us to take a backup for rollbacks.  Thankfully we worked at a bank, so that wasn't too late into the evening.  

Before the operations person could kick off their process we had to wait for a DBA to run all the scripts.  Aside from the dev team doing the deployment, a DBA and operations person would have to be online.  A script could take a second or 20 minutes to finish.  

In a nutshell, we had all this automation, except on the most crucial part of the application.  The exact same code was tested multiple times as it moved through environments.  A unique database delta script was manually created per environment.  There was no source of truth for the database schema.  An index might exist in pre-production but not QA.  Which environment one was right?  Meanwhile the DBAs are pulling their hair out trying to keep everything running.

To put it mildly, database change control was the wild west.

  