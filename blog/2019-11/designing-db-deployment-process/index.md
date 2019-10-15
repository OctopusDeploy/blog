---
title: How To Design a Automated Database Deployment Process
description: This article will walk you through how to design your ideal automated database deployment process 
author: bob.walker@octopus.com
visibility: public
published: 2019-11-08
metaImage: 
bannerImage: 
tags:
 - Engineering
 - Database Deployments
---

I went from deployments taking 2-4 hours down to 10 minutes.  In order to do that I had to automate the database deployments.  That didn't happen overnight.  It took some time and I made several mistakes.  The biggest mistake I made was at the beginning, by focusing exclusively on the tooling.  I wanted to automate the manual steps I had to do within the existing process.  I didn't realize how broken the existing process was.  The focus should've been on Developers and DBAs working together to come up with the ideal process.  Then worry about the tooling.  In this article I will walk you through how to design your ideal automated database deployment process.

!toc

## Overview of Designing Your Process

In this section I am going to provide a general overview of the steps to follow when designing your process.  In the next section I am going to walk through how a small team and I followed those steps.  Before diving in, I want to focus on some things to avoid:

- Don't spend weeks or months trying to design the perfect process.  Have a small group of people spend a day or two working on a draft.
- Don't focus on specific tooling.  Focus on what tooling in general can do.  For example, modern version control software supports branching along with some sort of review process for merging.  Every database deployment tool has a way to run it from the command line.  
- Don't try to automate rollbacks at the start.  You can spend the entire two days trying to figure out all the possible scenarios.  
- Don't be afraid to ask for help if you get stuck.  When I was working for a previous company we asked Redgate for help and they walked us through this process.  There are consulting firms, such as [DLM Consultants](http://dlmconsultants.com/), which can help.

Let's get this out of the way.  Your existing process works.  But you have come to the conclusion there are problems with it.  Write down or diagram your process.  Include everything from when a developer makes a change to promoting that change through environments until it gets to production.  

While writing the process was written down, focus on answering these questions.

1. Who are the people involved in the process?
2. What permissions do they have?
3. Why are they involved?
4. Which environments have a different process?
5. Why are they different?
6. What happens when the script fails to run?
7. Who reviews the scripts?
8. When are they reviewed?
9. Who needs to be involved with each deployment?
10. What isn't working and what needs to change?

With those answers in hand, it is time to start working on a rough draft of the ideal process.  When designing the process, don't mention specific tools.  Say `all database changes will be reviewed by a database developer during the merge process` instead of `all database changes will be reviewed by a database developer via a pull request prior to merging into master`.   

Keep in mind, this is a rough draft of the ideal process.  As you work on implementing it you will learn more and iterate.  And, the ideal process is the end goal.  Getting to the ideal process will involve a lot of changes.  You will be "moving a lot of cheese."  Expect to get push back.  Databases are the key component of most applications.  A bad script can result in ruined days or weeks.

It is important to build trust in your process.  

I've found the two best ways to build trust in the process is manual verification and pilot teams/applications.  For example, generate the delta script and have a DBA review prior to deploying to QA.  A pilot application or pilot team is great at helping prove out the process.  They can work with the group who drafted the process to find the tooling to implement it and deploy to Production.  Doing so finds the pain points in the proposed process and allows everyone to iterate on the process quickly.  

When it comes time for other teams to implement the process, they can include similar manual verification steps.  

To quickly summarize:

1. A small group of key people (developer, DBAs, etc) meet for 1 to 2 days.
2. They write down the current process.
3. Identify key people, pain points, requirements, and other important items.
4. Draft up an ideal process.
5. Have a pilot team or application research the tooling and implement the process to deploy to production.
6. Iterate on the process.
7. Decide on the tooling.
8. Start wide adoption.

## Tooling

Prior to joining Octopus I worked for a company where we spent months debating TeamCity vs Jenkins.  The primary argument was "Jenkins is Free" vs "TeamCity costs money but you also get these additional features."  10+ developers met multiple times for an hour each to debate this.  If you look at cost per developer hour, we could've bought a [100 agent TeamCity license](https://www.jetbrains.com/teamcity/buy/#new) with enough left over for pizza and beer party for all 100+ developers.  

The CI/CD tools today have a lot of features.  It is very easy to get lost trying to do side by side comparisons of feature sets.  Identify two or three critical features a tool must have.  That should narrow down the list.  After that, compare other items such as ease of use and learning curve.  

When I helped implement automated database deployments at previous companies the requirements were it must integrate with SQL Server Management Studio (SSMS), it should be able to pick up changes made to a database automatically, and it should be able to check in those changes to git.  95% of all developers at the company used SSMS to make database changes and several developers didn't have the background or knowledge to hand write schema change scripts.  Those two requirements filtered out tools such as DbUp, SSDT, and RoundhousE.  

In some cases, a tool meets two out the three critical requirements.  But a tool is in consideration because it is free.  It is hard to argue with free.  There are some questions to consider when a free tool is considered that meets 2/3 of the requirements vs a paid tool which meets 3/3 requirements.  

1. How much slower will adoption be because of that missing feature?  
2. Is there something which can be done to augment the free tool to get to 2.5/3 requirements met?  
3. Has the company attempted to use that free tool in the past?  Why wasn't it adopted?

I had a similar "free vs paid" debate with Redgate vs SQL Server Data Tools for Visual Studio (SSDT).  To purchase Redgate for 100+ developers it would cost well into six figures while SSDT is free.  But SSDT integrates with Visual Studio, not SSMS.  Several teams attempted to adopt SSDT in the past and all of them eventually abandoned the effort.  Too many people preferred to make their changes in SSMS not Visual Studio.  They ended up with a weird multi-step process to get the changes into SSDT.  I'm not saying SSDT is a bad tool.  I'm saying that it didn't work for that particular company.

## Real-World Use Case

It is one thing to talk about high level concepts.  It is another thing to put them into practice.  In this section I am going to walk you through how we used the above steps to automate our database deployments.  In doing so, we went from 2 hour deployments down to 10 minute deployments.

### The Existing Process

I stated at the beginning of the article I didn't realize how broken the process was.  It took until we reached out for help from Redgate, and they met with myself, a database developer, a DBA, and a database architect for two days to see how broken it was.

Our process was:

1. Developer makes change in `Development`.  All Developers have sysadmin rights in `Development`.
2. Developer changes connection in SSMS and makes change to `Test`.  All Developers have sysadmin rights in `Test`.
3. Database Developer or Lead Developer runs Redgate Schema Compare to generate delta script for `Test` and `Pre-Prod`.  Any complex database changes (move columns, combine columns, etc) are removed and manually scripted.  Scripts are saved to a shared folder.  Everyone except DBAs have read-only rights to `Pre-Prod`.  Database Developer or Lead Developer reviews changes at this time.
4. DBAs are notified via email to run scripts in shared folder on `Pre-Prod`.  They run the scripts and send output to requested.  They review all scripts which run on `Pre-Prod` during this time.
5. Multiple changes can be pushed to `Pre-Prod` prior to going to `Production`.  Database Developer or Lead Developer runs Redgate Schema Compare to generate delta script for `Pre-Prod` and `Production`.  Just like before, any complex database changes (move columns, combine columns, etc) are removed and manually scripted.  Scripts are saved to a shared folder.  Everyone except DBAs have read-only rights to `Production`.



FCSA and Telvent Mashup
- Dev admin rights
- Shared database model
- Drop scripts into shared folder with links in a doc
- Hand scripts over to DBAs to run
- Failure, we re-evaluate the scripts and make corrections
- Run again

### Roadblocks and why they are there

### Key people

### Requirements

### Ideal Process

### Short Term Plan

### Long Term Plan

### Iterations

## Conclusion