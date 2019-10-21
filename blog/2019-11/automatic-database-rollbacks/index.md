---
title: Pitfalls with Rollbacks with Automated Database Deployments
description: While it is possible to rollback a database, the real question is should you?  This article will walk through some 
author: bob.walker@octopus.com
visibility: public
published: 2019-11-14
metaImage: 
bannerImage: 
tags:
 - Engineering
 - Database Deployments
---

It is possible to configure automatic rollbacks with database deployments.  The real question becomes, should you?  This article will answer that question.  TL;DR; you shouldn't.  You should roll forward.  Read more to see why.

This article is one of many we have written on automated database deployments.  [Click here](https://octopus.com/database-deployments) to read the rest.

!toc

## Rollback Scenarios
Typically, the need to rollback will fall into one of three buckets.

- During deployment.
- During verification after a deployment.
- After deployment and verification.

Deployment and verification are intentionally split up.  It is possible for users to be using the application during verification.  It all depends on the deployment strategy (outage, canary, blue/green deployments) and what your application supports.  For example, Octopus Deploy has a maintenance mode.  This prevents other non-admins from deploying code, but they can still access it.

Let's go through a common application's deployment process.  The application has a database, a front end written in React, an RESTful API back-end and a background service.  The deployment process might look like:

1. Approve deployment to production.
2. Deploy database changes.
3. Deploy background service.
4. For each web server.
    1. Take server out of load balancer.
    2. Deploy RESTful API.
    3. Deploy Front End.
    4. Run sanity check tests.
    5. Add server back into load balancer.
5. Verify release by running full suite of tests.
6. Notify appropriate people of release.

For some deployments, it might be a new index or a efficiency tweak to a stored procedure.  Rolling those changes back would have little impact.  Rollback of schema changes, adding a column, moving a column, adding a table, splitting a table, would have significant impact.  Including a migration script makes rollbacks exponentially more complex.

## What Triggers a Rollback?
The short answer as to what triggers a rollback is "it depends."  Eventually, the discussion turns to rollbacks when helping customers automate their database deployments.  The two lines I hear the most are:

- When something goes wrong with the deployment, the process should automatically rollback.
- We should be able to run a series of tests, and if those tests fail, then everything should be rolled back.

Okay.  Cool.  "Something goes wrong" is a bit vague.  What exactly is the trigger or triggers?  The issue is deployment failures can happen for a variety of reasons.  

- Failure because of a network issue, a network admin restarted a switch and that killed the connection to the deployment target.
- A DBA applies a role membership change, but the new role doesn't have data reader permissions.
- The deployment target is turned off.
- The service account used to do deployments had its password changed earlier in the day.  But the deployment is still using the old password.
- The updated application requires .NET Core 3.0, but only .NET Core 2.2 is installed.
- The deployment process is misconfigured.

I don't think any of those would justify rolling back a database change.  They can be resolved fairly quickly and the deployment is retried.  

Running a series of tests and rolling back on failure is also vague.  Do ALL tests need to pass?  Just like deployment failures, tests can fail for a variety of reasons.

- Integration test causes application to call out to external system.  The external system isn't scheduled to be updated tomorrow and returns an unexpected result.  The result is correct for the current version of the external system.  But it is incorrect for the version going out tomorrow.
- Someone accidentally changes the names of some test customers.  The tests were expecting "Testing User" but now the data is "Testing User #1."
- An external system is being updated the same time your application is.  Tests are going to fail until that external system is up and running in 20 minutes.

Again, I don't think any of those failures justify rolling back a database change.  It is common for integration tests to only work when the stars align.  

The volume of changes being deployed can also influence a rollback.  If a couple of changes are being deployed and a failure occurs, then it might make sense to rollback.  But if several dozen changes are deployed during a scheduled outage window, and there is a bug with one, then it doesn't make a lot of sense to rollback.  Especially if the bug can be fixed in a patch later which doesn't require an outage window.  

## The Problem With Backups

You might be thinking to yourself, "why don't we take a database backup prior at the start of the deployment process?  That'll solve the problem of rolling back those complex changes."  A backup's usefulness has a limited lifespan.  A backup cannot be used for rollbacks the moment a user starts using the new version of the application.  That could be 1 minute or several hours.  Rolling back would mean changing data.  Or, even worse, a rollback would mean removing data.  

Rollback via a backup restore doesn't occur in a vacuum.  It is very common for applications to share data.  Prior to working at Octopus I worked at a bank.  It wasn't exactly a bank, it was closer to a credit union, but for all intents and purposes it was a bank.  I worked on the loan origination system, whose primary purpose was to gather data to send to a decision engine.  The decision engine would determine if the loan should be denied, auto approved, or needed a loan officer needed to gather more data.  Once the loan officer had more data entered they could submit it for subsequent decisions.  The loan origination system and decision engine were separate applications with separate databases.  The decision engine handled the first decision differently than the second decision.  It kept track of the decisions it was making based on a unique identifier sent in by loan origination system.  

The first time we restored the loan origination system from a backup in our test environment we started getting lots of unexpected results from the decision engine.  The issue was the decision's engine database wasn't restored from a backup.  We saw two issues main problems:

**Problem #1:**

1. User submits loan.
2. Decision engine auto approves loan.
3. Restore from backup occurs.
4. User submits loan again.
5. Decision engine sees this as a second request and rejects the loan thinking it is fraud.

**Problem #2:**

1. User submits loan for customer A.
2. Decision engine auto approves loan.
3. Restore from backup occurs.
4. The restore deleted the entire loan record, along with dozens of other loan records.
5. User recreates loan submits it, but it gets a different unique id to send to the decision engine.
6. The decision engine already has that unique id for Customer B.  It decides based off of Customer A and B's information.  

As you can see, backups shouldn't be used for rollbacks.  They should be used for disaster recovery only.  Typically the backups are rolling.  A full backup once a week, partial backup once a day with point in time backups every 15 or so minutes.  They are done in the event a database server or a data center is lost.

## The Problem With Rollback Scripts

I've seen some companies institute the rule "for every database change made, a corresponding rollback script must be written."  Rollback scripts have their own pitfalls.  

For starters, they have to be written.  Much more importantly, they have to be tested.  Who tests them?  When are they tested?  Is it an automated test or a manual test?  If the answers to those questions are, "Bob tests them at the morning before a production release" then chances are they aren't being tested.  If it is not a blocking task and if it is not automated then it won't get done.  

Rollback scripts have a limited lifespan just like database backups.  When the rollback is non-breaking, a new index is removed or a tweaked stored procedure is reverted, the lifespan is long.  When the change is breaking, a new column or table is removed, the lifespan is limited.  For example, a column is added to a table.  The rollback script removes the column.  Easy peasy.  But that column was in production for two days and hundreds of users saved data to that column.  Should the rollback script still remove that column?  Probably not, as removing data is a big deal in most applications.

The temptation is there to come up with a decision tree on when a rollback script is necessary.  In the end, that will make things worse.  Scenario after scenario after scenario will be added to the tree.  It will grow into an unwieldy mess.  

## Why Roll Forward Is The Answer

I've seen first-hand what a game changer automated database deployments are.  Deploying the "full stack" used to take 2-4 hours now takes 10-20 minutes.  The database portion of the deployment used to take 20-50 minutes now takes 2-3 minutes.  It is possible to get a fix to production in less than 30 minutes.  That includes build time and deploying to all three other environments.  

In 15 years, I've done thousands of deployments and I was involved in two rollbacks.  And one of those rollbacks was aborted at the last minute.  We started the discussion to rollback, and spent hours figuring out what would happen if we rolled back the code and database.  At the last minute I found the solution to the bug which triggered the rollback discussion.  Everyone breathed a collective sigh of relief because the rollback, as one person on the team told me later, would've been a "total disaster."  By the time we were done with our analysis with the rollback, a full day had passed since the code was deployed to production.  At this company, we had to have a rollback plan in place prior to going to production, but the first step was "analyze the impact of the rollback to the customer."  

The time I did an actual rollback involved rolling back the code only.  The business owner of the application told us that rolling back the database was not an option.  But the application was crashing over and over again.  We had to rollback, tell the users to only work in a specific area of the application (it was a internal application).  Two hours later we pushed a fix up and the application was back on track.  During the two hours a big chunk of the application had some funky errors.  But we knew what the issue was, we just needed some time to verify it.

The point is, it took us much, much longer to analyze, discuss, and plan the rollback, than it did to fix the actual issue.  

## Conclusion

There are only so many hours in the day.  Spend the time getting better at deployments rather than spending the time worrying about rollbacks and all the possible scenarios.  