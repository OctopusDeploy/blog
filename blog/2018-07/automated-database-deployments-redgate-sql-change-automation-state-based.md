---
title: Automated Database Deployments using State Based Redgate SQL Change Automation
description: Automated Database Deployments using State Based Redgate SQL Change Automation
author: bob.walker@octopus.com
visibility: private
published: 2018-07-16
tags:
 - Database Deployments
---

## Introduction

Previous blog posts discussed why [you need automated database deployments](https://octopus.com/blog/automated-database-deployments-series-kick-off) and [tips on getting started](https://octopus.com/blog/automated-database-deployments-iteration-zero) down that path.  Enough talk, it is time for action!  This article will walk you through setting up an automated database deployment pipeline using the [state based approach](https://www.red-gate.com/products/sql-development/sql-change-automation/approaches) for [Redgate's SQL Change Automation](https://www.red-gate.com/products/sql-development/sql-change-automation/).  I picked this tool to start with because it is easy to setup, integrates nicely with SSMS, and...well...I already had a demo setup.  I'm also a [little biased](https://www.red-gate.com/hub/events/friends-of-rg/friend/BobWalker) towards Redgate's tooling.  So there's that.

The end goal of this article is for you to have a working proof of concept for you to demo to your team or leaders within your organization.

## Prep Work

For this demo you will need a SQL Server instance running, an Octopus Deploy instance and a CI server.  I recommend using a dev instance or your local machine for this proof of concept.  

## Tools Needed

If you would like to follow along at home you will need the following.  The examples given will be using TeamCity and VSTS/TFS.  The examples will still apply for Jenkins and Bamboo, the UI will look different.

- Octopus Deploy
    - Get 45-day free trial for on-premise [here](https://octopus.com/licenses/trial).
    - Get 30-day free trial for Octopus Cloud [here](https://octopus.com/account/register).
- Redgate SQL Toolbelt
    - Get 14-day free trial [here](https://www.red-gate.com/dynamic/products/sql-development/sql-toolbelt/download).  
- CI Tool (pick one)
    - Jenkins - download [here](https://jenkins.io/download).
    - TeamCity - download [here](https://www.jetbrains.com/teamcity/download/).
    - TFS - download [here](https://visualstudio.microsoft.com/tfs/).
    - VSTS - try [here](https://visualstudio.microsoft.com/team-services/).
    - Bamboo - download [here](https://www.atlassian.com/software/bamboo/download)
- SQL Server Management Studio (SSMS)
    - Download for free [here](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms)
- SQL Server
    - SQL Express - download [here](https://www.microsoft.com/en-us/sql-server/sql-server-editions-express)
    - SQL Developer - download [here](https://www.microsoft.com/en-us/sql-server/sql-server-downloads)

## Installing Software

Most of the setup is pretty straight-forward.  Follow the wizards and the prompts and what-not.  

### Developer Machine
This is the machine where we will be making the schema changes and checking them into source control.  Redgate's SQL Toolbelt will prompt to install quite a bit.  

You only need to install the following:
- SQL Source Control
- SQL Prompt (it isn't required, but it makes your life so much easier)
- SSMS Integration Pack

### Build Server

Both Octopus Deploy and Redgate have plug-ins support for the major build servers.  For ease of use I have included the download links below.  

- Jenkins
    - Octopus - download [here](https://download.octopusdeploy.com/octopus-tools/4.37.0/OctopusTools.4.37.0.zip) - Please note, you can have Jenkins interact with Octopus by using octo.exe.  You can read more about that [here](https://octopus.com/docs/api-and-integration/jenkins)
    - Redgate - download [here](https://plugins.jenkins.io/redgate-sql-ci)
- TeamCity
    - Octopus - download [here](https://download.octopusdeploy.com/octopus-teamcity/Octopus.TeamCity.zip)
    - RedGate - download [here](https://www.red-gate.com/dlmas/TeamCity-download)
- VSTS/TFS
    - Octopus - download [here](https://marketplace.visualstudio.com/items?itemName=octopusdeploy.octopus-deploy-build-release-tasks)
    - Redgate - download [here](https://marketplace.visualstudio.com/items?itemName=redgatesoftware.redgateDlmAutomationBuild)
- Bamboo
    - Octopus - download [here](https://marketplace.atlassian.com/apps/1217235/octopus-deploy-bamboo-add-on?hosting=server&tab=overview)
    - Redgate - download [here](https://marketplace.atlassian.com/apps/1213347/redgate-dlm-automation-for-bamboo?hosting=server&tab=overview)

### Deployment Target
It is not recommended to install an Octopus Tentacle directly on SQL Server.  The [documentation](https://octopus.com/docs/deployment-examples/sql-server-databases#SQLServerdatabases-Tentacles) goes into further details why.  Instead we will be install the tentacle on a jumpbox which sits between Octopus Deploy and SQL Server.  For security you have two options, you can use integrated security, which will require you to set the tentacle to run as an service account or user account.  Here is [some documentation](https://octopus.com/docs/infrastructure/windows-targets/running-tentacle-under-a-specific-user-account) on how to configure that.  

For the jumpbox you will need to install the following items:
- SQL Change Automation PowerShell 3.0
- SQL Change Automation

## Sample Project

For this walk-through I modified the RandomQuotes project used in previous Will It Deploy videos.  If you haven't had the chance to check them out you are missing out.  Do yourself a favor and watch them.  Each episode is around 15 minutes.  You can find the playlist [here](https://www.youtube.com/playlist?list=PLAGskdGvlaw13QRF-ypT9h83QTPutlbMn).  

The source code for this sample can be found [here](https://github.com/OctopusDeploy/AutomatedDatabaseDeploymentsSamples).  You will need to fork this repository so you can make modifications to it later in this article.

## Configuring the CI/CD Pipeline

Everything you need is already checked into source control.  All we need to do is build it and push it out to the SQL Server.  

### Octopus Deploy Configuration

You will need the step templates from Redgate to [create a database release](http://library.octopus.com/step-templates/c20b70dc-69aa-42a1-85db-6d37341b63e3/actiontemplate-redgate-create-database-release) and [deploy a database release](http://library.octopus.com/step-templates/7d18aeb8-5e69-4c91-aca4-0d71022944e8/actiontemplate-redgate-deploy-from-database-release).  When you are browsing the step template you might notice the step template [to deploy directly from a package](http://library.octopus.com/step-templates/19f750fb-2ce8-4361-859e-2dfcdf08a952/actiontemplate-redgate-deploy-from-package).  How the state based functionality for SQL Change Automation Works is it will compare the state of the database stored in the NuGet package with the destination database.  Each time it runs it creates a new set of delta scripts to apply.  Because of that the recommended process is:

1) Download the database package onto the jump server.
2) Create the delta script.
3) Review the delta script (can be skipped in dev).
4) Run the delta script from the Jump Server on SQL Server.

Using the step template to deploy directly from the package prevents the ability to review the scripts.  

This is the process I have put together for deploying databases.

![](octopus-database-deployment-overview.png)

I am a firm believer of having tools handle all the manual work for me.  This is why my process will create the main SQL user for the database, the database, and add the SQL user to the database and the user to the role.  You can download those step templates directly from the Octopus Community Step Template Library.  If you are starting out giving up that much control can be scary.  Completely understandable.  The main steps you need from the above screen shot are:

![](octopus-database-required-steps.png)

Let's go ahead and walk through each one.  The download a package step is very straight forward, no custom settings aside from picking the package name.

![](octopus-database-download-package.png)

The Redgate - Create Database Release step is a little more interesting.  What trips up most people is the Export Path.  The export path is where the delta script will be exported to.  The "Redgate - Deploy from Database Release" step needs access to that path.  

![](octopus-create-database-release-step.png)

What I like to do is use a project variable.  

![](octopus-database-project-variables.png)

The full value of the variable is: 

```
    C:\RedGate\#{Octopus.Project.Name}\#{Octopus.Release.Number}\Database\Export 
```

Other recommendations on this screen:
- You will also notice I have supplied the username and password.  I recommend using integrated security and having the tentacle running as a specific service account.  However, I don't have Active Directory setup on my test machine so SQL Users it is.
- Take a look at the [default SQL Compare Options](https://documentation.red-gate.com/sr1/using-sql-compare-options-in-sql-release/default-sql-compare-options-used-by-sql-release) and make sure they match up with what you want.  If they don't then you will need to supply the ones you want in the "SQL Compare Options (optional)" variable.  You can view the documentation [here](https://documentation.red-gate.com/sc11/using-the-command-line/options-used-in-the-command-line).  If you do decide to go the route of custom options I recommend creating a  variable in a [library variable set](https://octopus.com/docs/deployment-process/variables/library-variable-sets) so those options can be shared across multiple projects.
- Use a custom filter in the event you want to limit what the deployment process can change.  I wrote a lengthly blog post on how to do that [here](https://www.codeaperture.io/2016/10/13/using-sql-source-control-to-filter-out-unwanted-items/).  My personal preference is to filter out all users and let the DBAs manage them or let Octopus manage them since it can handle environmental differences.

The next step is approving the database release.  I recommend creating a custom team to be responsible for this.  My preference is to skip this step in Dev and QA.  

![](octopus-approve-database-changes.png)

The create database release step makes use of the artifact functionality built into Octopus Deploy. This allows the approver to download the files and review them. 

![](octopus-database-artifacts.png)

The final step is deploying the database release.  This step takes the delta script in the export data path and runs it on the target server.  This is why I recommend putting the export path in a variable.

![](octopus-deploy-database-release.png)

That is it for the Octopus Deploy configuration.  Now it is time to move on to the build server.

### Build Server Configuration

For this blog post I will be using VSTS/TFS and TeamCity.  The build should, at a minimum, do the following:

1) Build a NuGet package containing the database state using the Redgate plug-in
2) Push that package to Octopus Deploy using the Octopus Deploy plug-in
3) Create a release for the package which was just pushed using the Octopus Deploy plug-in
4) Deploy that release using the Octopus Deploy plug-in

#### VSTS / TFS Build

Only three steps are needed in VSTS / TFS to build and deploy a database.  

![](vsts-build-database-overview.png)

The first step will build the database package from source control.  The items highlighted are the ones you need to change.  The subfolder path variable is relative.  I am using sample Git repo which is why I have that additional "RedgateSqlChangeAutomationStateBased" folder.

![](vsts-build-database-package.png)

The push package to Octopus step can be a little tricky.  You need to know the full path to the artifact generated by previous step.  I'm not 100% sure how you would know without trial and error.  

![](vsts-push-database-package.png)

Here is the full value in case you wish to copy it.

```
    $(Build.Repository.Localpath)\RandomQuotes-SQLChangeAutomation.1.0.$(Build.BuildNumber).nupkg
```

The Octopus Deploy Server must be configured in VSTS / TFS.  You can see how to do that by going to our [documentation](https://octopus.com/docs/api-and-integration/tfs-vsts/using-octopus-extension).

The last step is to create a release and deploy it to dev.  After connecting VSTS / TFS with Octopus Deploy you will be able to read all the project names.  You can also configure this step to deploy the release directly to Dev.  Clicking the "Show Deployment Progress" will stop the build and force it to wait on Octopus to complete. 

![](vsts-create-octopus-database-release.png)

#### TeamCity

The TeamCity setup is very similar to the VSTS/TFS setup.  Only three steps are needed.

![](teamcity-build-sql-automation-overview.png)

The first step, the build database package step, has similar options to VSTS/TFS.  You will need to enter in the folder as well as the name of the package.

![](teamcity-redgate-build-database.png)

The kicker is you have to enter in a package version in the advanced options.  If you don't then you will start getting random errors from the Redgate tooling saying something about an invalid package version.

![](teamcity-redgate-build-advanced-options.png)

The publish package step requires all three of the options to be populated.  By default the Redgate tool will create the NuGet package in the root working directory.

![](teamcity-publish-package.png)

The final step is creating and deploying the release.  Very similar to before, you provide the name of the project, the release number and the environment you wish to deploy to.

![](teamcity-create-database-release.png)

## The CI / CD Pipeline In Action
Now it is time to see all of this in action.  For this demo I will be creating a new database, RandomQuotes_BlogPost_Dev.

![](octopus-first-database-release-variables.png)

As you can see I do not have any databases with that name.  You can see I have used this SQL Server as my test bench for automated deployments.

![](ssms-before-deployment.png)

Let's take a quick look at the tables stored in source control.  

![](github-tables-to-be-created.png)

If we open up one of those files we can see the create script generated by Redgate's SQL Source Control.

![](github-database-quote-table.png)

Kick off a build and let's see that whole pipeline in action.  The build looks successful.

![](vsts-build-first-time.png)

No surprise, the deployment was successful in Octopus Deploy.  The VSTS/TFS build was set to wait on Octopus Deploy to finish deploying the database.  If the deployment failed the build would've failed. 

![](octopus-first-database-release.png)

Going back to SSMS and we can now see the database and the tables have been created.

![](database-successful-deployment-ssms.png)

## Changing the Schema

Well that is all well and good but that was a project already created.  Let's make a small change and test the whole process.  There is a bit more setup involved with doing this.

1) Clone your forked repo to your local machine.
2) Open up SSMS and create a random quotes database on your local machine or dev.
3) In SSMS bind the source controlled database to the newly created database.  You can read how to do that in the [documentation](https://www.red-gate.com/products/sql-development/sql-source-control/resources/how-to-set-up-sql-source-control)

When linking the database to source control you need to provide the full path to the folder where the source control is stored.  I store all my code in a folder called C:\Code.git.  So the full path is:

![](ssms-redgate-linked-database-path.png)

Here is the text of that path for you to copy:

```
C:\Code.git\AutomatedDatabaseDeploymentsSamples\RedGateSqlChangeAutomationStateBased\db\src\
```

Once you are finished you should see something like this:

![](ssms-redgate-successful-link.png)

Now we can make the change to the database.  For this test, let's just add in a stored procedure which will return a value.

![](ssms-sample-sproc.png)

Now we can commit that change to source control.

![](ssms-sample-commit.png)

And assuming the CI/CD build is set to fire on commit you should see that new sproc appear in Dev!  

## Conclusion

Automating database deployments does require a bit of prep-work but the payoff is well worth the effort.  Having the auditing alone is well worth it.  With this tool I can now see who made a change, when a change was made and when that change went to production.  In the past that was kept in another location with a 50/50 shot of it being updated.

As you start down this journey my recommendation is to add the manual verification step to all environments until trust has been established.  This will ensure you don't accidentially check in a change which blows away half the team's database changes.  

Until next time, happy deployments!