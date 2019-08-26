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

## TODO: Replace intro with "real world story" from when I was a developer

Blue/Green deployments are near and dear to my heart.  It is something I've sort of done in the past, but I ran into some interesting hiccups.  If you're not familiar with blue/green deployments, it is when you have two identical production environments labeled blue and green.  At any given time, only one of those environments, for example, blue, is live.  Deployment is done to the non-live environment, for example, green, which is then verified.  After verification is complete, a switchover occurs, and the live environment becomes green.  

![](https://i.octopus.com/docs/deployment-patterns/blue-green-deployments/images/3278250.png)

There are several advantages to doing this.  Rollbacks should be easy, only a matter of switching from blue to green or green to blue.  When a switchover does occur, it should be seamless as the code has already been running, there is no need to wait for it to compile or "warm-up."  And changes can be verified in production without having any customers hitting the code, making the deployment much less risky.  If something doesn't work, don't make the switch, try again at a later time.

As someone who was a developer for 15 years, that is what I wanted.  With that, I could do the middle of the day deployments.  No more signing on during weird hours of the night.  What always tripped me up was the database.  For the vast majority of applications I've seen and worked on, there is a single database.  In this post, I will walk through some techniques you can use to do blue/green deployments with a database.  What I've tried, what works, and what falls flat.

!toc

## How Database Changes Add Complexity

Let's start with a what seems like a simple example to see how database changes add complexity.  A new column `CustomerFullName` is added, which is a combination of `CustomerFirstName` and `CustomerLastName` columns.  In the UI, there used to be two fields, `First Name` and `LastName`, but the decision was made to combine the two fields.  Some cultures have multiple names which do not fit into the `First Name` and `Last Name` we commonly see in the US.  A migration script was written to backfill `CustomerFullName` with `CustomerFirstName` and `CustomerLastName` during the deploy.  

If we didn't have Blue/Green deployments, there would be a concern that customers could get onto the site while the verification process is occurring.  And there will be a small window in which the database changes will be deployed, but the code won't be deployed.  I've seen companies solve this by deploying after hours or disabling access to users.  Sadly, for the majority of the companies, I've worked for previously after hours meant late nights or weekends.  The worst was 2:00 AM on Saturday.  Ugh.

1. Disable access to users or turn off the website.
2. Run the script to add the column `CustomerFullName`
3. Run the migration script to populate `CustomerFullName`
4. Deploy the code to `Production`
5. Verify the code in `Production`
6. Profit

Blue/Green deployments solve the problem of users accessing the system while you are attempting to verify the code.   But that increases complexity for the database change.  Specifically around the migration script.  It could run after the column is added and before the code is deployed.  This way, you can run the necessary tests to make sure the name is appearing and saving correctly.  There will be a time frame between when the migration script originally started and when verification is complete.  It could be a few minutes; it could be a half-hour or more.  During that time, new records are added, and existing ones are updated.  The migration script will have to be run again after the environments are swapped.  

1. Run the script to add the column `CustomerFullName`
2. Run the migration script to populate `CustomerFullName`
3. Deploy code to `Blue` environment
4. Verify the code and database changes in the `Blue` environment
5. Swap the live environment from `Green` to `Blue`
6. Run the migration script to populate `CustomerFullName`

It could be a few seconds or a few minutes to from swapping the live environment from `Green` to `Blue` to when the migration script finishes running a second time.  Running a SPA App ([Single Page Applications](https://en.wikipedia.org/wiki/Single-page_application)) adds even more fun.  They will need to have their UI refreshed before the new field shows up.  In the meantime it will be sending API requests without the `CustomerFullName` field, just the `CustomerFirstName` and `CustomerLastName` field.  

Finally, the `CustomerFirstName` and `CustomerLastName` columns will need to be deleted.  Having obsolete columns in a table is just asking for trouble.  When should that clean-up occur?  Who keeps track of cleaning up unused items in the database?  If we weren't doing Blue/Green deployments the migration script could delete the column after it was finished.  But that would put you in an spot where rollbacks would be impossible.  

## Scenarios and Recommendations.

Admittedly, that scenario is fairly complex.  Most database changes I've seen don't combine two columns into one and try to backfill the new column.  But it happens, and you should be prepared for it.  The TL;DR; of my recommendation on how to handle that is don't have a migration script, everything should be handled in the code.  I'll get to my reasoning behind that in a bit, but first I want to walk through much easier scenarios and some recommendations on how to solve them.

**Please Note:** These are recommendations only.  There is no way I can cover every possible change you can make to a database.  My goal is to provide you with something which you can then modify to meet your own needs.  

I'm intentionally excluding the tooling you can use to automate database deployments.  This post is intended to be a 1000 foot view.

### Scenario #1: Adding a new column

