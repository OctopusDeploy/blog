---
title: Pitfalls with rollbacks and automated database deployments
description: While it is possible to roll back a database, the real question is, should you?  This article will walk through some questions to consider.
author: bob.walker@octopus.com
visibility: public
published: 2019-11-14
metaImage:
bannerImage:
tags:
 - Engineering
 - Database Deployments
---

It is possible to configure automatic rollbacks with database deployments.  It’s also possible to configure manual rollbacks with database deployments.  The question is, should you?  Rolling back a database change isn’t as straightforward as rolling back a code change.  Databases are the lifeblood of applications.  Numerous pitfalls can result in bad data or deleted data.  This article will go through those pitfalls and why roll forwards is a better approach.  

:::success
**TL;DR**

Rolling forward is a better option.  In specific instances, it is possible to roll back a database change, but those cases are rare.  The effort spent on rolling back databases should instead focus on making deployments as fast and safe as possible.  A fast and safe database deployment allows you to roll forward.
:::

This article is one of many we have written on automated database deployments.  [Click here](https://octopus.com/database-deployments) to read the rest.

!toc

## Rollback scenarios

The need to rollback will typically fall into one of three categories:

- During deployment.
- During verification after a deployment.
- After deployment and verification.

Deployment and verification are intentionally split up because users could be using the application during verification.  It all depends on the deployment strategy (outage, canary, or blue/green deployments) and what your application supports.  For example, Octopus Deploy has the [maintenance mode](https://octopus.com/docs/administration/managing-infrastructure/maintenance-mode) feature, which prevents other non-admins from deploying code, even though they can still access it.

Let’s go through a typical application’s deployment process.  The application has a database, a front end written in React, a RESTful API back-end, and a background service.  The deployment process might look like:

1. Approve the deployment to production.
2. Deploy database changes.
3. Deploy background service.
4. For each web server:
    1. Take the server out of the load balancer.
    2. Deploy the RESTful API.
    3. Deploy the Front End.
    4. Run sanity check tests.
    5. Add the server back into the load balancer.
5. Verify the release by running a full suite of tests.
6. Notify the appropriate people of the release.

For some deployments, it might be a new index or an efficiency tweak to a stored procedure, and rolling back those changes would have little impact.  Rollback of schema changes, adding a column, moving a column, adding a table, or splitting a table would have a significant impact.  Including a migration script makes rollbacks exponentially more complex.

## What triggers a rollback?

The two most common answers, I hear the most are:

1. Something goes wrong with the deployment, and the process should automatically rollback.
2. We run a series of tests, and if those tests fail, everything should be rolled back.

Okay, cool, but *something goes wrong* is a bit vague, so what exactly is the trigger or triggers?  Deployment failures can happen for a variety of reasons, for instance:

- Failure because of a network issue. Maybe a network admin restarted a switch, and that killed the connection to the deployment target.
- A DBA applies a role membership change, but the new role doesn’t have data reader permissions.
- The deployment target is turned off.
- The service account that is used for deployments had its password changed earlier in the day, but the deployment is still using the old password.
- The updated application requires .NET Core 3.0, but only .NET Core 2.2 is installed.
- A misconfiguration in the deployment process.

I don’t think any of those would justify rolling back a database change.  Often a retry can resolve those issues.

The second reason, running a series of tests and rolling back on failure is also vague.  Do **all** tests need to pass?  Just like deployment failures, tests can fail for a variety of reasons, for instance:

- The integration test causes the application to call out to the external system.  The external system isn’t scheduled to be updated until tomorrow and returns an unexpected result.  The result is correct for the current version of the external system, but it is incorrect for the version going out tomorrow.
- Someone accidentally changed the names of some test customers.  The tests were expecting **Testing User**, but now the data is **Testing User #1**.
- An external system is updated at the same time as your application.  Tests are going to fail until that external system is up and running 20 minutes from now.

Again, I don’t think any of those failures justify rolling back a database change.  It’s common for integration tests to only work when the stars align.

The volume of changes being deployed can also influence a rollback.  If a couple of changes are being deployed and a failure occurs, then it might make sense to rollback.  Several dozen changes deployed during an outage window with a single bug makes it hard to justify a rollback.  In that case, it could make sense to issue a patch later.  

That is why we added the [guided failure feature](https://octopus.com/docs/deployment-process/releases/guided-failures) to deployments in Octopus Deploy.  It allows you to choose how to handle a deployment failure.  

## The problem with backups

You might be thinking to yourself, “Why don’t we take a database backup before the start of the deployment process?  That’ll solve the problem of rolling back those complex changes.”  A backup’s usefulness has a limited lifespan.  A backup cannot be used for rollbacks the moment a user starts using the new version of the application.  That could be after 1 minute or several hours.  Rolling back would mean changing data.  Or, even worse, a rollback would mean removing data.

Rollback via a backup restore doesn’t occur in a vacuum.  It is very common for applications to share data.

Before working at Octopus Deploy, I worked on a loan origination system, with the primary purpose of gathering data to send to a decision engine.  The decision engine determined if the loan should be denied, auto-approved, or needed a loan officer to gather more data.  After the loan officer had entered more data, they could submit it for subsequent decisions.  The loan origination system and decision engine were separate applications with separate databases.  The decision engine handled the first decision differently than the second decision.  It kept track of the decisions it was making with a unique identifier sent in by the loan origination system.

The first time we restored the loan origination system from a backup in our test environment, we started getting lots of unexpected results from the decision engine.  The issue was the decision engine’s database wasn’t restored from a backup.  We saw two problems:

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
5. The user recreates the loan and submits it, but it gets a different unique ID to send to the decision engine.
6. The decision engine already has that unique ID for Customer B, and it decides based on Customer A and B’s information.  

As you can see, backups shouldn’t be used for rollbacks.  They should be used for disaster recovery only.  Typically, rolling backups happen with a full backup once a week, partial backup once a day, and point in time backups every 15 or so minutes.  These backups are done in the event a database server or a data center is lost.

## The problem with rollback scripts

I’ve seen some companies institute the rule: *For every database change made, a corresponding rollback script must be written*. However, rollback scripts have their pitfalls.  

Writing rollback scripts takes time.  If the script is never run, then it turns out to be wasted time.  If you have several dozen successful deployments, the motivation to write the rollback script will decrease as time goes on, to the point where people start openly questioning why it needs to be created in the first place.

Much more importantly, the scripts have to be tested.  That leads to further questions, such as:

- Who tests them?  
- When are they tested?  
- Is it an automated test or a manual test?  

If the answer to those questions is, *Bob tests them in the morning before a production release*, then there’s a chance the tests will happen only half the time.  If it is not a blocking task, and if it is not automated, then it won’t get done.  

Rollback scripts have a limited lifespan, just like database backups.  When the rollback is non-breaking, for instance, a new index is removed, or a tweaked stored procedure is reverted, the lifespan is long.  When the change is breaking, for instance, a new column or table is removed, the lifespan is limited.  

For example, if a column is added to a table, the rollback script removes the column.  Easy.  But if that column was in production for two days, and hundreds of users saved data to that column, should the rollback script still remove that column?  Probably not, as removing data is a big deal in most applications.

There is a temptation to come up with a decision tree on when a rollback script is necessary.  In the end, that will make things worse.  Scenario after scenario after scenario will be added to the tree, and it will grow into an unwieldy mess.  

## Rolling forward: A different way of thinking

Once a user starts using the application after a deployment, you have [crossed the Rubicon](https://en.wikipedia.org/wiki/Crossing_the_Rubicon), and depending on the database change made and the application, once verification starts, you could have also crossed the Rubicon.  Time and time again, I have seen the effort to **_successfully_** rollback a deployment, exceed the effort it would take to fix the issue, and push it to production.  

Rollbacks are a last resort.  So much so that when the topic is broached during a post-deployment crisis, the team will have to:

- Analyze what changes went out with the previous deployment.  Often it involves opening up a diff tool and going through the changes line by line.
- Create a list of areas to test to ensure the application doesn’t crash after the rollback.
- Identify which data is going to be changed.
- Identify which data is going to be deleted.
- Determine if the database needs to be rolled back or just the code.
- Create a backup of production and restore it on a test server.  
- Rollback the previous deployment and start testing.  

After database deployments were automated for the loan origination system, a typical deployment took 10 minutes.  Database only changes typically took 2-4 minutes.  We had four environments, and when we knew what the issue was, we could check in a change, verify it, and have it pushed to all four environments in less than 30 minutes.  If we wanted to skip the two lower environments, it was less than 20 minutes.  

## The ideal solution: rolling forward and staging backward-compatible changes

Database deployments are often the riskiest part of a deployment, so you need to take steps to minimize the risk.  

I think back to all the production deployments I’ve done, and the least stressful deployments were the ones without database changes, which is not always possible.  Deployments where the database changes were staged hours or days before the code was deployed were also low stress.  That gave us time to verify the changes during working hours.  The code was still deployed during the deployment window, but that typically went very quickly, and the verification was quicker.

That’s only possible when the database changes are backward compatible, and that takes a great deal of discipline. For details on how backward-compatible database changes should be made, see my article on [Blue/Green Database Deployments](https://octopus.com/blog/databases-with-blue-green-deployments).  That example is a bit extreme, but I would argue that the time spent on it was worth it.  

This approach also means the process of deploying database changes needs to be separate from the process that deploys code changes.  It does require a bit more planning, but it gives you the advantage of being able to roll back to a previous version of the code if a failure occurs during code deployment or verification. The chances of you needing to roll back the code after a database change is staged are much smaller.  Even then, the window for rollbacks is pretty small.  The previous version of the code can run with the latest database changes, but a rollback could end up removing newly added functionality, which leads to frustrated users.

It’s not always feasible to do that.  My personal rules are:

- When possible, make database changes backward compatible.
- Stage database changes in production hours or even days before the code deployment.
- Unless something catastrophic happens, roll forward.

## Conclusion

The chance of something going wrong with a rollback is much higher than when rolling forward.  That is especially true for database rollbacks.  There are only so many hours in the day, and it’s better to spend the time getting better at deployments than spending time worrying about rollbacks and all the possible scenarios.  

Happy Deployments!

If you enjoyed this article and would like to see more posts on automated database deployments please [click here.](https://octopus.com/database-deployments)
