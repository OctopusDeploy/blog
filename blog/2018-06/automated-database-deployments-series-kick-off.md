---
title: "Automated Database Deployments Series Kick Off"
description: Automated Database Deployments Series Kick Off
author: bob.walker@octopus.com
visibility: public
published: 2018-06-04
tags:
 - Database Deployments
---

The most nerve-racking part (for me) for any deployment is the database.  Deploying code is far less stressful.  If something wasn't right the code could be rolled back.  By the time it got to production, there should be zero surprises.  It is same code which had been tested in dev, and QA, and pre-production.    

Databases are not as flexible.  Imagine there is an error in a script and the names of all the users get deleted.  There isn't a good way to roll that back.  Sure a backup could be restored.  But when was that backup taken?  Have any users been in the system since that backup?  What data will be lost if the backup is restored?

## A Manual Process
Before working at Octopus Deploy I was a lead developer on a loan origination system for a large financial institution in the United States.  At any given point hundreds of millions of dollars of loans were in flight.  These were large loans requiring many individuals to work on it.  Each person might spend anywhere from 15 minutes to an hour working on the loan.  A financial officer might spend a half hour entering customer's financials.  An underwriter could spend an hour researching the customer and placing any notes and conditions on the loan.  Needless to say, restoring a database backup was the last resort.  Telling them they have to re-enter the information can happen only so many times before they would want to hunt me down.

At this time we had automated code deployments.  The database delta scripts were manually created by a database developer before the deployment.  It was common to see them spending a full business day or two putting the scripts together.  That manual creation also meant very little testing could occur.  When the script ran in production it was most likely the first time it has ever been run.  Sure we used tools like Redgate's SQL Compare.  We also tested on restored backups when possible.  But that could only do so much.  

Due to the risk, deployments could only occur after hours.  That meant evenings or weekends.  This allowed us to take a backup for rollbacks.  Thankfully, we worked at a financial institution, so that wasn't too late in the evening. Before the operations person could kick off their process to deploy the code we had to wait for a DBA to run all the scripts.  Aside from the dev team doing the deployment, a DBA and operations person would have to be online.  A script could take a second or 20 minutes to finish.  

Because of scheduling and risk, we could only deploy major releases once a quarter.  With minor bug fixes done between.  And we typically had several bug fix releases after a major release.  The majority of the time it was because some schema change was missed.  Deploying once a quarter meant a lot of changes were put in all at once.  Verification of the changes took quite a bit of time.  It was very common to see each release take 2-3 hours.

A schema change was missed because there was no source of truth for the database.  An index might exist in pre-production but not QA.  Which environment was right?  When was that index added?  Who added it?  To compound that the database had close to 6000 objects in it (mostly CRUD stored procedures).  The poor database developer had to resort to manually keeping track of all the changes.  80% of the time the database developer would make the change.  The other 20% of the time a developer would make the change.  Maybe the database developer was out sick that day.  We tried to remember to email them the change.  Think about all the changes you made in the last quarter to your code.  Do you remember them all?  

In a nutshell, we had all this automation, except on the most crucial part of the application.  The exact same code was tested multiple times as it moved through environments.  A unique database delta script was manually created per environment.  There was no source of truth for the database schema. Meanwhile, the DBAs are pulling their hair out trying to keep everything running.

To put it mildly, database change control was the wild west.  To summarize our challenges:

- Environment was all different
- Custom scripts were created per environment
- Database Objects existed in one environment and not the other
- Deployments took hours
- "Big bang" quarterly releases with multiple bug fix releases
- Manual tracking of changes

## Automating Database Deployments
Something had to give.  We had to get better.  We couldn't deliver value fast enough.  Database changes had to be in source control in some way.  Those scripts needed to be packaged up and automatically ran during a deployment.  After a lot of discussion, research, and testing we landed on a tool.  The tool itself isn't important.  What was important is we automated the database deployments. 

The impact was almost immediately noticeable.

Having the database in source control allowed us to see when a person made a change.  We were able to tie it back to stories.  Now we knew why a change was made.  And we set it up so if the change wasn't in source control then it would be deleted.  An index is in QA but not in source control?  It got deleted.  It was a little harsh but it guaranteed the database schema matched what is in source control.

The emergency fixes because some random view or stored procedure was missed dropped to near zero.  We set up a manual intervention step in Octopus Deploy which allowed us to approve database changes prior to a deployment.  Anything that looked was unexpected and we stopped the deployment.  That helped the DBAs trust the automated deployments.

The confidence in the deployments started increasing.  Soon we were doing deployments once a month.  Then once a week.  Features could be delivered to the user as soon as they passed verification by QA and a business owner.  A bug could be reported, it could be in the user's hands as soon as it passed verification.  Because we released so frequently the number of changes made decreased significantly.  The deployments went from taking 2-3 hours to 5-20 minutes.

Putting it all in Octopus Deploy had a side benefit of giving the DBAs and operations team their nights and weekends back.  Now they could schedule a deployment. They would have to get online only if something went wrong. 

## A New Blog Series
So you might be asking yourself, why in the world are you telling me all of this?  What is the point of all of this?  To pat your own back?  Well...maybe a little.   Really it is to announce I'm kicking off a blog series where I'll walk through the process of setting up database lifecycle management (DLM) and database deployment automation.  

The goal of the series is to provide you with some real-world examples using a variety of database deployment tools.  In addition to that, we will discuss some common pitfalls you will run across.  Excited to get started!