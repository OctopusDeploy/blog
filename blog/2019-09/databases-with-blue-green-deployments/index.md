---
title: Automated Database Blue/Green Deployments 
description: Learn some techniques on how to automate database deployments when using a blue/green deployment strategy.
author: bob.walker@octopus.com
visibility: public
metaImage: 
bannerImage: 
published: 2019-09-30
tags:
 - Database Deployments
---

I've been seeing more and more customers looking at adopting a blue/green deployment pattern.  If you're not familiar with blue/green deployments, it is when you have two identical production environments labeled blue and green.  At any given time, only one of those environments, for example blue, is live.  A deployment is done to the non-live environment, for example green, which is then verified.  After verification is complete a switchover occurs and the live environment becomes green.  

![](https://i.octopus.com/docs/deployment-patterns/blue-green-deployments/images/3278250.png)

There are several advantages to doing this.  Rollbacks should be easy, only a matter of switching from blue to green or green to blue.  When a switchover does occur it should be seamless as the code has already been running, there is no need to wait for it to compile or "warm up."  And changes can be verified in production without having any customers hitting the code, making the deployment much less risky.  If something doesn't work, don't make the switch, try again at a later time.

As someone who was a developer for 15 years, that is exactly what I wanted.  With that I could do middle of the day deployments.  No more signing on during weird hours of the night.  What always tripped me up was the database.  For the vast majority of applications I've seen and worked on, there is always a single database.  In this post, I will walk through some techniques you can use to do blue/green deployments with a database.  

!toc

## General Deployment Process

Before diving into database techniques, let's first look at what a generic process for Blue/Green deployments looks like.

For this example, let's assume Green is the live environment and Blue is the inactive environment.

1. Deploy code to the Blue environment
2. Verify code in the Blue environment
3. Swap the live environment from Green to Blue

Now, we are going to add the database into the mix.  That is when things get a little more interesting.  Same use case, Green is active, Blue is inactive.

1. Deploy the schema changes
2. Optional - Run any migration scripts to backfill data
3. Deploy code to Blue environment
4. Verify the code and database changes in the Blue environment
5. Swap the live environment from Green to Blue
6. Optional - Run any migration scripts to backfill data

You'll notice the migration scripts are run twice.  There is a reason for that.  We will use the example of adding a new column, called "CustomerFullName" to a Customers table.  The code being deployed will no longer have a separate field for FirstName and LastName.  The CustomerFullName column will be initially populated using the "FirstName" and "LastName" column.  After that the customers can change the value in that column through the UI.  The first time the migration script runs it will populate all the existing records.  Verification of the code changes in Blue could take several minutes or more.  1 to N number new customer records could be added during that time.  The second time the migration script runs should take a few seconds.

1. Deploy the schema changes
2. Run any migration scripts to backfill data - update 10,000,000 rows takes roughly 4-5 minutes
3. Deploy code to Blue environment
4. Verify the code and database changes in the Blue environment
5. Swap the live environment from Green to Blue
6. Run any migration scripts to backfill data - update 5000 rows takes roughly 1-3 seconds