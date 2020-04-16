---
title: Building trust in an automated database deployment process
description: It is not possible to automate database deployments without trust.  This post will walk through techniques on how to build trust in your automated database deployment process.  
author: bob.walker@octopus.com
visibility: public
published: 2099-01-01
metaImage: 
bannerImage: 
tags:
 - Engineering
 - Database Deployments
---

No matter the company, it's the same story.  A SQL script ran during a deployment caused a production outage.  Trust is lost and processes are created in an attempt to prevent future slip-ups.  More often than not, these are manual verification steps.  It's unrealistic to think nine women making a baby in a month; it's ludicrous to believe manual verifications will prevent 100% problems from reaching production.  Automation is needed.  But that lack of trust is pervasive.  If we can't trust people, how can we trust software?  This post will walk through some common strategies to build that trust back up.

!toc

## Digging into the lack of trust

Ask any DBA their greatest fears with the databases their responsible for and you'll probably hear the same two answers:

- `Production` Outage
- Data Loss

DBAs will move heaven and earth to prevent that from happening.  It's why production databases are running in a cluster or always on availability groups with the databases saved on a RAID array in a SAN with regular backups enabled and not on a single VM up in AWS.  

A database deployment could cause both of those problems.  A developer writing a SQL script to "clean-up data" could forget a where clause in the delete statement.  A script to move data from one table to another could run out of order and a drop column command is ran too soon.  

I experienced this first hand.  I worked for a company which had four environments, `Dev` -> `Test` -> `Pre-Prod` -> `Prod`.  Developers had admin rights in `Dev` and `QA`.  `Pre-Prod` and `Prod` deployments were done by the DBAs.  

`Test` had 10x more data than `Dev`, with more test cases being added by QA.  To leverage all that great data during development, all developers, myself included, pointed our machines to `Test`.  Quite often a database change was made days or weeks prior to the corresponding code change being pushed to `Test`.  To get the code to work, 1 to N changes could be made on `Test`.  As a result, the changes didn't go through a review process until it was time to go to `Pre-Prod`.  `Pre-Prod` was deployed to 2 to N number of times for every `Prod` deployment.  The script to deploy to `Pre-Prod` was completely different than `Prod`.  For added fun, the list of database changes was stored on a piece of paper, or in a Trello board or an Excel file.  We'd use Redgate's SQL Compare and uncheck all the changes not on the list.  

To summarize:
1. No review process prior to pushing a change to `QA`.
2. Reviews don't happen for days or weeks.
3. Manual process to create delta script between `QA` and `Pre-Prod` and `Pre-Prod` and `Prod`.  A change not ready to be pushed could easily slip through.
4. The `Prod` script is never tested prior to running on `Prod`.
5. Manually updating the list of pending changes.  Some changes in `Test` could go to `Pre-Prod` while others had to wait.  Often times the list was old or out of date.
6. No history of schema changes.

DBAs would comb over any proposed schema changes.  But they didn't have any context on how an application should work and what changes _should_ be included.  It was very easy for column to be missed in a view or stored procedure.  Fixing the view or stored procedure doesn't take long, but it still requires an emergency fix.  

It wasn't just the DBAs who didn't trust us, now the business didn't trust us.  

## The hidden cost of zero trust

No data loss.  No `Production` outages.  Easy to say, much harder to execute.  A manual deployment presents the chance for both.  New features and bug fixes need to be deployed.  To minimize the impact policies such as these are enacted.

1. One deployment per quarter.
2. Deployments occur off-hours only.
3. DBAs must review all database changes.

Ironically, these policies result in more risk, not less risk.  A lot of changes can be made in a single quarter.  The list can get so big no single person can remember everything which needs to go out.  During the off-hours deployment more verification steps are added to ensure nothing is missed.  Deployments take hours.  Despite everyone's best efforts, some random database change is missed resulting in an emergency fix the next day.  That emergency further erodes trust.  

It becomes an endless cycle.

## Building trust

Trust doesn't magically happen because of automation.  A delete script without a where clause will still cause data loss.  Automation only means a service account runs the script instead of a DBA.  

However, manual reviews result in a lot of noise.  A database delta script might only modify a where clause in a stored procedure.  Without source control, it can take quite a bit of time to review the change.  Somehow a diff needs to be generated to review the differences.  Do you open up script along with the `Prod` version in a tool like Beyond Compare?  That requires saving the `Prod` to a file so Beyond Compare can use it.













The default Blackberry ringtone gives me an adrenaline rush.  Back in late 2009/2010 I was on a four-person on-call rotation which handed around a Blackberry phone.  A page was sent anytime one of the 300+ production SQL Job failed.  All the SQL Jobs ran SSIS packages maintained by another team.  A quarter into the rotation and my trust in that team had completely eroded.