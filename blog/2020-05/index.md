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

When I started automating database deployments I was afraid the tooling would drop a column or table when it shouldn't.  I couldn't help but always wonder, did I have everything configured correctly?  The core problem is I didn't include the necessary steps to build trust in my database deployment process.  In this blog post I will walk through some techniques and configurations I used to help build that trust.

!toc

## Article Scope

This article could talk about the entire database deployment process.  But that has already been done.  I've outlined an ideal database development process in an [earlier article](https://octopus.com/blog/designing-db-deployment-process).  [A follow-up article](https://octopus.com/blog/use-case-for-designing-db-deployment-process#draft-of-the-ideal-process) went into further deail.  

This article is going to focus on how to build trust in the deployment pipeline using database tooling with Octopus Deploy.  This article will work under the following assumptions:

1. Developers are using isolated or local databases for development work.
2. All changes are checked into a branch and are merged using a pull request or some sort approval process.
3. The build server has already:
    - Perform the necessary steps to ensure the database changes are syntactically correct.  
    - Any static analysis tools, such as SQL Enlight, are ran to find potential code smells.
    - Database unit tests (yes they exist!) and if possible, integration tests are run.
    - Packaged up the database changes into a zip file or NuGet package and pushed to Octopus Deploy.

## Octopus Deploy features 

Octopus Deploy provides a number of features this article will leverage.

- [Manual Interventions](https://octopus.com/docs/deployment-process/steps/manual-intervention-and-approvals) which will pause the deployment and wait for someone to approve or reject the change.
- [Artifacts](https://octopus.com/docs/deployment-process/artifacts) to store the delta scripts for the DBAs (or whomever) to review during the manual interventions.
- [Run Conditions](https://octopus.com/docs/deployment-process/conditions#run-condition) to always notify the development team of the deployment outcome and to page the DBAs on production failures.
- [Custom Log Levels](https://octopus.com/docs/deployment-examples/custom-scripts/logging-messages-in-scripts) used by the run a script step to notify DBAs of potential problems in the scripts.
- [Email and other notification options](https://octopus.com/docs/deployment-process/steps/email-notifications) to let the approvers know of a pending change.
- [Guided failure mode](https://octopus.com/docs/managing-releases/guided-failures) which will pause the deployment on failure and let people look into it and potentially try again.
- [Output variables](https://octopus.com/docs/projects/variables/output-variables) to enable skipping unnecessary steps in the deployment.  For example, if the tooling reports no database changes, there is no point in running the deploy database change step.
- [Audit Log](https://octopus.com/docs/administration/managing-users-and-teams/auditing) to know who to blame when something goes wrong.


