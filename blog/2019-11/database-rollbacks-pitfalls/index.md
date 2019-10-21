---
title: Pitfalls with Rollbacks with Automated Database Deployments
description: While it is possible to roll back a database, the real question is, should you?  This article will walk through some 
author: bob.walker@octopus.com
visibility: public
published: 2019-11-14
metaImage: 
bannerImage: 
tags:
 - Engineering
 - Database Deployments
---

It is possible to configure automatic rollbacks with database deployments.  It's also possible to configure manual rollbacks with database deployments.  The question then becomes, should you?  Rolling back a database change isn't as straightforward as rolling back a code change.  Databases are the lifeblood of applications.  Numerous pitfalls could result in bad data or deleted data.  This article will go through those pitfalls and why roll forwards is a better approach.  

```
TL;DR; It is much better to roll forward.  In specific instances, it is possible to roll back a database change.  However, those cases are rare.  The effort spent on rolling back databases should instead focus on making deployments as fast and as safe as possible.  A fast and safe database deployment allows you to roll forward.
```

This article is one of many we have written on automated database deployments.  [Click here](https://octopus.com/database-deployments) to read the rest.

!toc

## Rollback Scenarios
Typically, the need to rollback will fall into one of three buckets.

- During deployment.
- During verification after a deployment.
- After deployment and verification.

Deployment and verification are intentionally split up.  Users can be using the application during verification.  It all depends on the deployment strategy (outage, canary, blue/green deployments) and what your application supports.  For example, Octopus Deploy has the [maintenance mode](https://octopus.com/docs/administration/managing-infrastructure/maintenance-mode) feature.  Maintenance mode prevents other non-admins from deploying code, but they can still access it.

Let's go through a common application's deployment process.  The application has a database, a front end written in React, a RESTful API back-end, and a background service.  The deployment process might look like:

1. Approve deployment to production.
2. Deploy database changes.
3. Deploy background service.
4. For each web server.
    1. Take the server out of the load balancer.
    2. Deploy the RESTful API.
    3. Deploy Front End.
    4. Run sanity check tests.
    5. Add the server back into the load balancer.
5. Verify release by running a full suite of tests.
6. Notify appropriate people of release.

For some deployments, it might be a new index or an efficiency tweak to a stored procedure.  Rolling those changes back would have little impact.  Rollback of schema changes, adding a column, moving a column, adding a table, splitting a table would have a significant impact.  Including a migration script makes rollbacks exponentially more complex.

## What Triggers a Rollback?
The short answer as to what triggers a rollback is "it depends."  The two lines I hear the most are:

- When something goes wrong with the deployment, the process should automatically rollback.
- We should be able to run a series of tests, and if those tests fail, then everything should be rolled back.

Okay, cool.  "Something goes wrong" is a bit vague.  What exactly is the trigger or triggers?  The issue is deployment failures can happen for a variety of reasons.  

- Failure because of a network issue, a network admin restarted a switch, and that killed the connection to the deployment target.
- A DBA applies a role membership change, but the new role doesn't have data reader permissions.
- The deployment target is turned off.
- The service account used to do deployments had its password changed earlier in the day.  But the deployment is still using the old password.
- The updated application requires .NET Core 3.0, but only .NET Core 2.2 is installed.
- A misconfiguration in the deployment process.

I don't think any of those would justify rolling back a database change.  Often a retry can resolve those issues.

Running a series of tests and rolling back on failure is also vague.  Do ALL tests need to pass?  Just like deployment failures, tests can fail for a variety of reasons.

- The integration test causes the application to call out to the external system.  The external system isn't scheduled to be updated tomorrow and returns an unexpected result.  The result is correct for the current version of the external system.  But it is incorrect for the version going out tomorrow.
- Someone accidentally changes the names of some test customers.  The tests were expecting "Testing User," but now the data is "Testing User #1."
- An external system is updated at the same time your application is.  Tests are going to fail until that external system is up and running in 20 minutes.

Again, I don't think any of those failures justify rolling back a database change.  It is common for integration tests to only work when the stars align.  

The volume of changes being deployed can also influence a rollback.  If a couple of changes are being deployed and a failure occurs, then it might make sense to rollback.  Several dozen changes deployed during an outage window with a single bug makes it hard to justify a rollback.  In that case, it could make sense to issue a patch later.  

That is why we added the [guided failure feature](https://octopus.com/docs/deployment-process/releases/guided-failures) to deployments in Octopus Deploy.  It allows you to choose how to handle a deployment failure.  

## The Problem With Backups

You might be thinking to yourself, "why don't we take a database backup before the start of the deployment process?  That'll solve the problem of rolling back those complex changes."  A backup's usefulness has a limited lifespan.  A backup cannot be used for rollbacks the moment a user starts using the new version of the application.  That could be 1 minute or several hours.  Rolling back would mean changing data.  Or, even worse, a rollback would mean removing data.  

Rollback via a backup restore doesn't occur in a vacuum.  It is very common for applications to share data.  Before working at Octopus Deploy, I worked at a bank.  It wasn't exactly a bank, it was closer to a credit union, but for all intents and purposes, it was a bank.  

I worked on the loan origination system, whose primary purpose was to gather data to send to a decision engine.  The decision engine would determine if the loan should be denied, auto-approved, or needed a loan officer needed to gather more data.  Once the loan officer had more data entered, they could submit it for subsequent decisions.  The loan origination system and decision engine were separate applications with separate databases.  The decision engine handled the first decision differently than the second decision.  It kept track of the decisions it was making based on a unique identifier sent in by the loan origination system.  

The first time we restored the loan origination system from a backup in our test environment, we started getting lots of unexpected results from the decision engine.  The issue was the decision's engine database wasn't restored from a backup.  We saw two issues problems:

**Problem #1:**

1. The user submits a loan.
2. The decision engine auto approves the loan.
3. Restore from backup occurs.
4. The user submits the loan again.
5. The decision engine sees this as a second request and rejects the loan thinking it is a fraudulent loan.

**Problem #2:**

1. The user submits the loan for customer A.
2. The decision engine auto approves the loan.
3. Restore from backup occurs.
4. The restore deleted the entire loan record, along with dozens of other loan records.
5. User recreates loan submits it, but it gets a different unique id to send to the decision engine.
6. The decision engine already has that unique id for Customer B.  It decides based on Customer A and B's information.  

As you can see, backups shouldn't be used for rollbacks.  They should be used for disaster recovery only.  Typically the backups are rolling.  A full backup once a week, partial backup once a day with a point in time backups every 15 or so minutes.  They are done in the event a database server or a data center is lost.

## The Problem With Rollback Scripts

I've seen some companies institute the rule "for every database change made, a corresponding rollback script must be written."  Rollback scripts have their pitfalls.  

Writing a rollback scripts takes time.  If the script is never run, then it turns out to be wasted time.  If you have several dozen successful deployments, the motivation to write the rollback script will decrease as time goes on.  To the point where people start openly questioning why it needs to be created.

Much more importantly, they have to be tested.  That leads to further questions, such as who tests them?  When are they tested?  Is it an automated test or a manual test?  If the answers to those questions are, "Bob tests them in the morning before a production release," then chances are the tests happen half the time.  If it is not a blocking task and if it is not automated, then it won't get done.  

Rollback scripts have a limited lifespan, just like database backups.  When the rollback is non-breaking, a new index is removed, or a tweaked stored procedure is reverted, the lifespan is long.  When the change is breaking, a new column or table is removed, the lifespan is limited.  For example, a column is added to a table.  The rollback script removes the column.  Easy peasy.  But that column was in production for two days, and hundreds of users saved data to that column.  Should the rollback script still remove that column?  Probably not, as removing data is a big deal in most applications.

The temptation is there to come up with a decision tree on when a rollback script is necessary.  In the end, that will make things worse.  Scenario after scenario after the scenario will be added to the tree.  It will grow into an unwieldy mess.  

## Rolling Forward - A Different Way of Thinking

Once a user starts using the application after a deployment, you have [crossed the Rubicon](https://en.wikipedia.org/wiki/Crossing_the_Rubicon).  Depending on the database change made and the application, once verification starts, you have crossed the Rubicon.  Time and time again, I have seen the effort to **_successfully_** rollback a deployment exceed the effort it will take to fix the issue and push it to production.  

Rollbacks are a last resort.  So much so that when the topic is broached during a post-deployment crisis, the team will have to:

- Analyze what changes went out with the previous deployment.  Often it involves opening up a diff tool and going through the changes line by line.
- Create a list of areas to test to ensure the application doesn't crash after rollback.
- Identify which data is going to be changed.
- Identify which data is going to be deleted.
- Does the database need to be rolled back or just the code?
- Create a backup of production and restore it on a test server.  Rollback the previous deployment and start testing.  

Once database deployments were automated for the loan origination system, a typical deployment took 10 minutes.  Database only changes typically took 2-4 minutes.  We had four environments.  Once we knew what the issue was, we could check in a change, verify it, and have it pushed to all four environments in less than 30 minutes.  If we wanted to skip the two lower environments, it was less than 20 minutes.  

## The Ideal Solution - Rolling Forward and Staging Backward Compatible Changes

Database deployments are often the riskiest part of a deployment.  Steps should be taken to minimize the risk.  

I think back to all the production deployments I've done.  The least stressful deployments were the ones without database changes, which is not always possible.  Deployments where the database changes were staged hours or days before the code was deployed was also low stress.  That gave us time to verify the changes during working hours.  The code is still deployed during the deployment window.  But that typically went very quickly.  Also, the verification was quicker.

But that is only possible when the database changes are backward compatible.  That takes a great deal of discipline.  My article on [Blue/Green Database Deployments](https://octopus.com/blog/databases-with-blue-green-deployments) detailed how backward-compatible database changes should be made.  That example was a bit extreme.  I would argue that the time spent is worth it.  

It also means the process of deploying database changes needs to be separate from the process which deploys code changes.  It does require a bit more planning.  The advantage to that is if a failure did occur during code deployment or code verification, it becomes possible to roll back to a previous version of the code.  The chances of you needing to roll back the code after a database change is staged are much smaller.  Even then, the window for rollbacks is pretty small.  The previous version of the code can run with the latest database changes, but a rollback could end up removing newly added functionality, which leads to frustrated users.

It is not always feasible to do that.  My rules of thumb are:

- When possible, make database changes backward compatible.
- Stage database changes in production hours or even days before the code deployment.
- Unless something catastrophic happens, roll forward.

## Conclusion

The chance of something going wrong with a rollback is much higher than rolling forward.  That is especially true for database rollbacks.  There are only so many hours in the day.  It is better to spend the time getting better at deployments rather than spending time worrying about rollbacks and all the possible scenarios.  

Happy Deployments!

If you enjoyed this article and would like to see more posts on automated database deployments please [click here.](https://octopus.com/database-deployments)