---
title: Rollbacks with Octopus Deploy
description: Learn about how to implement a rollback strategy with Octopus Deploy, without having to implement advanced deployment patterns.
author: bob.walker@octopus.com
visibility: public 
published: 2021-10-27-1400
metaImage: 
bannerImage: 
bannerImageAlt: 
isFeatured: false
tags:
 - Product
---

Whenever the topic of rollbacks comes up, the conversation turns to blue/green, red/black, or canary deployment patterns. Those patterns make rollbacks much easier, however, they're time intensive to implement, and sometimes they're not necessary. Maybe you pushed out an API change to Test, and you want to get back to a known good state.  That's not when you should attempt to implement those patterns for the first time.  

In this post, I walk through a rollback strategy you can execute today, without implementing advanced deployment patterns.

:::hint
**Out of scope**
Rolling back database changes is out of scope for this article because doing them successfully [is a complex topic with many pitfalls](https://octopus.com/blog/database-rollbacks-pitfalls).  This post focuses on code-only rollbacks. It demonstrates how to skip database deployment steps during a rollback.  In practice, code and UI changes are much more frequent than database changes, especially in the test environment.  Most schema changes happen at the start of a new feature, with minor tweaks during testing.
:::

# What is a rollback?

This post will help you modify your existing (working and tested) deployment process to support rollbacks.  

First, let's examine what a rollback is trying to accomplish.  

Consider these scenarios:

- The QA team is blocked because of a bug introduced in a recent deployment test, and the fix is hours from being checked in.
- During a production deployment verification, a show-stopping bug is found that will require over a day to fix and test.

The goal is the same in both scenarios; quickly return the application to a known good state.

:::hint
Ironically, many customers are focused on the production scenario, yet the test scenario occurs more often, and has a bigger impact than most people realize.  If you follow Octopus Deploy's core rule of [build once, deploy everywhere](https://octopus.com/blog/build-your-binaries-once), the chance of a show-stopping bug making to **Production** is rare (but not impossible).  However, test is different; I've seen the mindset that only a few people are affected.  That's untrue though, as deadlines will slip if QA is blocked for hours at a time.  
:::

The goal is to get back to a known good state, but that's different from a deployment.  Skipping specific steps can make rollback faster.  Many deployment processes are created with the assumption that none of the dependent software or infrastructure has been configured.  For example, a deployment process could trigger a runbook to create a database if it didn't already exist.  Or, install the latest version of Node.js.  During a rollback, those additional steps are not needed.  If you check to see if the database existed when deploying `2021.2.3` of your application, you won't need to check again when rolling back to `2021.2.1`.  

For this article: 

> A rollback is getting back to a known good state by running a modified version of the original deployment process.  

# Rolling forward or rolling back

Not all releases can and should be rolled back.  The scenarios above specifically mention a fix is hours or days away.  In many cases, rolling forward is typically less risky and time-consuming.  A minor fix can be easier to test and deploy than rolling back a major release.  

Here are some typical reasons why we recommend rolling forward.

- You cannot choose which piece of code to roll back in a binary.  It either all rolls back, or nothing rolls back.  A team on a once a month or once a quarter release schedule will have dozens if not 100s of changes.  That's why we also recommend smaller changesets released at a greater frequency.
- Often, database and code changes are tightly coupled together.  Safely rolling back a database without data loss is [extremely difficult](https://octopus.com/blog/database-rollbacks-pitfalls), if not impossible.
- Users will notice when something is changed and then changed back, especially for custom business applications used all day by the same user base.  
- With the proliferation of [service-oriented architecture](https://en.wikipedia.org/wiki/Service-oriented_architecture) (SOA) and its cousin [microservices](https://en.wikipedia.org/wiki/Microservices) code changes are rarely made in isolation.  "Proper" SOA and microservices architecture are loosely coupled to each other and their clients.  However, in the real world, coupling exists.  A rollback to a back-end service can have downstream impacts.

Despite that, there are several scenarios when a rollback can be the right solution.  A legacy monolith application with a large database can be successfully rolled back in specific circumstances.  Some of those scenarios include (but are not limited to):

- Styling or markup only changes
- Back-end code changes with no public interface or model changes
- Zero to minimal coupling with external services or applications
- Zero to minimal database changes (new index, changing a stored procedure for performance improvements, tweaked view including additional columns on already joined tables)
- Number of changes since the last release is small

While we recommend rolling forward, having a rollback process in place is a valuable option in your CI/CD pipeline, even if a rollback occurs once a month.

# Test your rollback process

About ten years ago, a few hours after a production deployment, I was informed about a show-stopping bug.  I was surprised as the release had gone through weeks of verification by QA.  We couldn't establish the cause and concluded a rollback was needed.  That was the first rollback in years.  The rollback plan defined in the deployment Word doc was always "rollback to the previous version of code."  That is more of a wish than a detailed plan.  Immediately the issue was escalated, causing everyone involved with the release to be paged (from QA, to business owners, and managers).  

A new rollback plan had to be created from scratch.  Even with a new plan, we pegged the odds of a successful rollback at 10%. It was a no-win situation. We had a show-stopping bug we couldn't repro (and therefore fix), or we could roll back and take our chances.  

Rolling back offered at least some chance as opposed to none (which is what we had in our current state). Each person was assigned a task. I went through the changelog item by item, and documented the impact of rolling back.  

Fifteen minutes before we made the final rollback decision, I discovered a small block of code that looked suspicious.  I ran tests to hit that block of code, and established it was causing the show-stopping bug.  We aborted the rollback plans, implemented a fix, and pushed it out later that day.  Everyone was relieved tat we didn't have to test an unproven rollback process.

I'm telling you this story because you should test your rollback processes multiple times.  In a perfect world, it should be tested and verified once a week.  During a production outage, the last thing you want to do is develop a new rollback process or run an untested rollback process.  

# Example Deployment Process

For the rest of this article, I will update an existing deployment process to support rollbacks.  I chose [OctoFX Sample Application](https://github.com/OctopusSamples/OctoFX) for this example because it is similar to a lot of applications I've seen (and worked on).  It has the following components:

- SQL Server Database
- Windows Service
- ASP.NET MVC Website

The deployment process for this application is:

1. Run a runbook to create the database when it doesn't exist.
1. Deploy the database changes.
1. Deploy the Windows Service.
1. Deploy the Website.
1. Pause the deployment and verify the application.
1. Notify stakeholders the deployment is complete.

![Original Windows Deployment Process](original-deployment-process.png)

Your database platform, back-end service, and front-end might be using completely different technology.  In this article, I will update the process to skip specific steps and run additional steps during a rollback.  

# Re-deploy previous release

The core concept of my rollback process will be re-deploying a previous release.  That can be done by:

Selecting the release you want to re-deploy to your target environment.  In my example, I am going to re-deploy `2021.9.9.3` to **Test**.

![Project Overview select previous release](project-overview-select-previous-release.png)

Click on the overflow menu and select **Re-deploy...**.

![Selecting re-deploy from overflow menu](overflow-redeploy-menu.png)

You'll be sent to the deployment screen.  Click the **DEPLOY** button to kick off the re-deployment.

![Deployment screen from the previous deployment](previous-version-deployment-screen.png)

# Deployment Mode

Re-deploying a previous release as-is means _all_ the steps from the previous deployment will be re-run.  As stated earlier, a rollback's goal is to get back to a known state by running a slightly modified deployment process.  

:::hint
Your rollback process will be different than the example; I'm using the database steps as the example.  The goal is to show you _how_ to disable the steps, rather than _what_ is being disabled.
:::

To disable specific steps for a rollback, we need to know a rollback is occurring.  But we are going to be re-deploying an existing release.  The tricky thing is re-deploying the same release to the current environment is a valid use case.  What we need to know is the "deployment mode."

- **Deployment**: First time a release is deployed to a specific environment to add new features, fix bugs, and more to the application.
- **Rollback**: Re-deploying a previous release in a specific environment to return to a known good state.
- **Re-deployment**: Re-deploying the same release in a specific environment when a new server comes online or you need to "kick" the application.

We need to know this because it changes the deployment process.  The deployment process during a deployment runs all the steps.

1. Run a runbook to create the database when it doesn't exist.
1. Deploy the database changes.
1. Deploy the Windows Service.
1. Deploy the Website.
1. Pause the deployment and verify the application.
1. Notify stakeholders the deployment is complete.

A rollback will skip the first two steps.

1. ~~Run a runbook to create the database when it doesn't exist.~~
1. ~~Deploy the database changes.~~
1. Deploy the Windows Service.
1. Deploy the Website.
1. Pause the deployment and verify the application.
1. Notify stakeholders the deployment is complete.

For my application, I only redeploy when the web farm is scaled out.  I never scale out the app servers or database.  So I only want to deploy the website and notify the stakeholders.  

1. ~~Run a runbook to create the database when it doesn't exist.~~
1. ~~Deploy the database changes.~~
1. ~~Deploy the Windows Service.~~
1. Deploy the Website.
1. ~~Pause the deployment and verify the application.~~
1. Notify stakeholders the deployment is complete.

What is needed is the ability to calculate the "deployment mode."  Octopus provides the necessary system variables to do that.

`Octopus.Release.Number`: The current release's number (`1.2.2`).
- `Octopus.Release.CurrentForEnvironment.Number`: The ID (`1.1.1`) of the last **successful** release deployed to the current environment.

To calculate "deployment mode" you'd compare `Octopus.Release.Number` with `Octopus.Release.CurrentForEnvironment.Number`.  If it is greater, then it is a **Deployment**; if it is less than then it is a **Rollback**, and if they are the same, then it is a **Re-deployment**.

# Calculate Deployment Mode Step Template

I've created the step template, [Calculate Deployment Mode](https://library.octopus.com/step-templates/d166457a-1421-4731-b143-dd6766fb95d5/actiontemplate-calculate-deployment-mode), to perform that calculation for you.  Using that result, it will set several output variables.

 - **DeploymentMode**: Will be `Deploy`, `Rollback`, or `Redeploy`.
 - **Trigger**: This indicates if the deployment was caused by a deployment target trigger or a scheduled trigger.  It will be `True` or `False`.
 - **VersionChange**: Will be `Identical`, `Major`, `Minor`, `Build`, or `Revision`.

While working on the step template, I realized almost everyone would want to use the **DeploymentMode** output variable in a [variable run condition](https://octopus.com/docs/projects/steps/conditions#variable-expressions).  Because of error handling, the syntax for the run condition can be tricky to get right.  Octopus will always evaluate the variable run condition to determine if the step should run, even if an error occurs in a previous step.  If we don't include error handling in the run condition, it could evaluate to `True` and run the step.  We don't want that. 

The variable run condition when the deployment mode is **Rollback** with all the necessary error handling is:

```
#{unless Octopus.Deployment.Error}#{if Octopus.Action[Calculate Deployment Mode].Output.DeploymentMode == "Rollback"}True#{else}False#{/if}#{/unless}
```

I added the following output variables with the necessary error handling and comparison logic to make that easier. 

- **RunOnDeploy**: Only run the step when the DeploymentMode is **Deploy**.
- **RunOnRollback**: Only run the step when the DeploymentMode is **Rollback**.
- **RunOnRedeploy**: Only run the step when the DeploymentMode is **Redeploy**.
- **RunOnDeployOrRollback**: Only run the step when the DeploymentMode is **Deploy** or **Rollback**.
- **RunOnDeployOrRedeploy**: Only run the step when the DeploymentMode is **Deploy** or ** Re-deploy**.
- **RunOnRedeployOrRollback**: Only run the step when the DeploymentMode is **Redeploy** or **Rollback**.
- **RunOnMajorVersionChange**: Only run the step when the VersionChange is **Major**.
- **RunOnMinorVersionChange**: Only run the step when the VersionChange is **Minor**.
- **RunOnMajorOrMinorVersionChange**: Only run the step when the VersionChange is **Major** or **Minor**.
- **RunOnBuildVersionChange**: Only run the step when the VersionChange is **Build**.
- **RunOnRevisionVersionChange**: Only run the step when the VersionChange is **Revision**.

With those output variables, the syntax for that same **Rollback** run condition is:

```
#{Octopus.Action[Calculate Deployment Mode].Output.RunOnRollback}
```

# Prevent Release Progression Step Template

Earlier I mentioned there would be an additional step I want to run during a rollback.  One example of such a step is to block the release progression.  A rollback, even in a **Test** environment, is a significant event.  If a release has a lot of bugs in it, you don't want it to move onto the next environment in the lifecycle.  

Octopus provides the ability to [prevent release progression](https://octopus.com/docs/releases/prevent-release-progression), however, that is a manual step.  I'm not a fan of manual steps, so I made a new step template, [Block Release Progression](https://library.octopus.com/step-templates/78a182b3-5369-4e13-9292-b7f991295ad1/actiontemplate-block-release-progression) to prevent the release progression as part of the deployment process.  

# Deployment Process with Rollback Steps

Using the [Calculate Deployment Mode](https://library.octopus.com/step-templates/d166457a-1421-4731-b143-dd6766fb95d5/actiontemplate-calculate-deployment-mode), [variable run condition](https://octopus.com/docs/projects/steps/conditions#variable-expressions), and [Block Release Progression](https://library.octopus.com/step-templates/78a182b3-5369-4e13-9292-b7f991295ad1/actiontemplate-block-release-progression), the updated deployment process is:

1. Calculate Deployment Mode
1. Run a runbook to create the database when it doesn't exist (only run when deployment mode is **Deploy**).
1. Deploy the database changes (only run when deployment mode is **Deploy**).
1. Deploy the Windows Service.
1. Deploy the Website.
1. Block Release Progression (only run when deployment mode is **Rollback**).
1. Pause the deployment and verify the application (only run when deployment mode is **Deploy** or **Rollback**).
1. Notify stakeholders the deployment is complete.

![Windows simple deployment process](windows-simple-rollback-process.png)

:::hint
Use the step notes feature to indicate which step runs during deployments, rollbacks, or always.

![Step notes feature](step-notes-feature.png)
:::

## Setting Run Conditions

The steps marked as "only run when deployment mode is **Rollback**" or "only run when deployment mode is **Deploy** or **Rollback**" will need the **Run Condition** updated to be a variable.  The variable will be one of the output variables from the [Calculate Deployment Mode](https://library.octopus.com/step-templates/d166457a-1421-4731-b143-dd6766fb95d5/actiontemplate-calculate-deployment-mode) step.  

![Variable run condition](variable-run-condition.png)

## Testing the rollback

For my test, I have two releases:

- `2021.9.9.5`: This is currently in the **Development** environment.
- `2021.9.9.6`: This is a new release I want to deploy to **Development**.

Deploying `2021.9.9.6` to **Development** went as expected.  Step 6 is skipped as that is set only to run when the deployment mode is **Rollback**.

![2021.9.9.6 deployed to dev](version-2021-9-9-6-to-dev.png)

In my test scenario, a showstopping bug in `2021.9.9.6` is found after the deployment.  We want to:

- Rollback to `2021.9.9.5`.
- Block progression on `2021.9.9.6` to prevent it from being deployed to **Test** or **Production**.

Re-deploying `2021.9.9.5` went as expected.  Steps 2 and 3 were skipped while step 6 ran.

![Rolling back to 2021.9.9.5](rollback-to-2021-9-9-5.png)

In addition, `2021.9.9.6` has had the release progression blocked. Users will see a visual indicator on the project dashboard.

![Project overview with release progression blocked](blocking-release-progression.png)

# Automatic Rollbacks

The next logical step is to think about triggering the rollback.  I'd recommend manually triggering the rollback and logging why you are doing it.  When you see a pattern, add in automated tests to detect if a specific condition is met.  The concern I have is getting a "false positive," and a release is rolled back in **Production** when it shouldn't have.  

For this scenario, I'd hold off on triggering the rollback automatically until you have automated all the steps to make a rollback decision.  For example, if one of your conditions has no database changes, you should have a script check the SQL Scripts for schema changes (add table, add column, etc.).  If a schema change is found, then a rollback isn't possible.

Once you have done that, then automatically trigger the rollbacks for all your non-production environments.  After several successful rollback decisions have been made, then it is time to turn it on for **Production**.  

# Wrapping Up

I'll be the first to admit it; I've been bullish on adopting advanced deployment patterns such as Blue/Green, Red/Black, or Canary as the only way to roll back.  That thought process went along with the mindset: you should only roll forward if you can't adopt those patterns.  To do so can cost a lot of time and money on existing applications.  Because of that, they should be adopted when there is a legitimate business reason to do so.  For example, Google can never have an outage, so a Canary-style deployment makes sense.  But an internal business application used by a few dozen people from 6 AM EST to 10 PM PST won't get the same benefits for the cost.

My previous "adopt advanced deployment patterns or roll-forward only" is an extreme point of view.  It doesn't have to be all or nothing.  You can create a rollback process with a few tweaks to your existing deployment process using [variable run condition](https://octopus.com/docs/projects/steps/conditions#variable-expressions) and the new step templates, [Calculate Deployment Mode](https://library.octopus.com/step-templates/d166457a-1421-4731-b143-dd6766fb95d5/actiontemplate-calculate-deployment-mode) and [Block Release Progression](https://library.octopus.com/step-templates/78a182b3-5369-4e13-9292-b7f991295ad1/actiontemplate-block-release-progression).  While it won't support every possible rollback scenario, it will give you another option when a bug is found.


## Register for the webinar: Rollback strategies with Octopus Deploy

A robust rollback strategy is key to any deployment strategy. In this webinar, we’ll cover best practices for IIS deployments, Tomcat, and full stack applications with a database. We’ll also discuss how to get the rollback strategy right for your situation. 

We're running 3 sessions of the webinar over 2 days, from Wed 4 Nov – Thurs 5 Nov, 2021.

<span><a class="btn btn-success" href="/events/rollback-strategies-with-octopus-deploy">Register now</a></span>

Happy deployments!