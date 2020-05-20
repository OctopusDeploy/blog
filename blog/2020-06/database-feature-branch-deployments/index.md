---
title: Database Feature Branch Deployments
description: With the Octopus API it is possible to clone almost everything needed in a space.
author: bob.walker@octopus.com
visibility: private
published: 2020-12-31
metaImage: 
bannerImage: 
tags:
 - Engineering
---

My [previous article](LINK PENDING) walked through how I've adjusted my thinking around feature branch deployments.  It's one thing to write thought experiments like that article.  It's another to put that into practice.  In this article I will walk through how I set up a database deployment process for feature branches.  

!toc

## Quick recap

My [previous article](LINK PENDING) comes in at over 2000 words.  The TL;DR; of that article is:

The static workflow of {{dev, test, staging, production}} does not work when using git.  Git makes it incredibly easy to create feature branches.  That static workflow essentially required everyone to check in their code to a trunk (master or development) to be properly tested.  That resulted in three problems:

1. Unfinished code makes it way into a trunk, and needs to be tested.
2. There is no clear path to production for bug fixes, requiring a work around, such as setting up a bug fix branch consisting of just {{staging, production}}.
3. It is common for 2 to N features to be worked on at the same time.  This means multiple merges of unfinished code into a trunk.  The chance of an incorrect merge conflict resolution is increased.

The following changes should be done to solve those issues:

1. Combine **Dev** and **Test** into one environment, **Test**.
2. Each feature branch should be given a separate sandbox in **Test**.
3. Once a feature branch has been tested and verified by QA, it should be merged into master.
4. The deployments for master start in **Staging**.  It never goes through test.
5. **Test** becomes a dynamic environment, with resources being added and removed as needed.  **Staging** and **Production** are static and are stable.

## Feature branch database deployments business rules

Naturally, there are a lot of questions when looking at those changes.    

- When will the feature branch sandbox be created?  
- Should it be a fresh database or should a backup be restored?  
- If it is a restored backup what should it be a back up of?  Production?  Staging?  
- How often should that backup be made?  
- Finally, when does that sandbox get torn down?

Those questions are only around the creation of the feature branch sandboxes in **Test**! There are a few other questions I typically encounter.  A lot of these questions are focused on building trust in the database deployment process.  

- When should the DBAs get involved?  **Production** is too late, and when the feature branch is created in **Test** is too early.
- Who should trigger the deployment to **Production**?
- Can the **Production** deployment be scheduled, only page the DBAs when something goes wrong?
- Can we see what is going to **Production** without doing a **Production** deployment?

For my process I made the following decisions.

1. Each deployment will check for the feature branch database, if it doesn't see it, create a new one.
2. When creating a feature branch database, restore a copy of **Staging**.
3. The **staging** backup will be created on each deployment to **Staging**.
4. Backup and then delete the feature branch database when it is merged into master.
5. The DBAs will approve all deployments to **Staging**.  
6. The **Staging** deployment will also generate a delta report for **Production**.  This way the DBAs approve a release once.
7. The DBAs will trigger the **Production** deployment.  They can either schedule it or start it right away.

