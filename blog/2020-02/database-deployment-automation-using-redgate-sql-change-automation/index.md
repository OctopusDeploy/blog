---
title: Database deployment automation using state-based Redgate SQL Change Automation
description: Setting up automated database deployments with state-based Redgate SQL change automation.
author: bob.walker@octopus.com
visibility: public
published: 2020-02-25
metaImage: metaimage-redgate-database.png
bannerImage: blogimage-redgate-database.png
bannerImageAlt: Database deployment automation using state-based Redgate SQL Change Automation
tags:
 - Engineering
 - Database Deployments
---

![Database deployment automation using state-based Redgate SQL Change Automation](blogimage-redgate-database.png)

My previous blog posts discussed why [you should consider automated database deployments](blog/2020-02/why-consider-database-deployment-automation/index.md) and [tips for getting started](blog/2020-02/database-deployment-automation-approaches/index.md).

This article walks through setting up a database deployment automation pipeline using the [state-based approach](https://www.red-gate.com/products/sql-development/sql-change-automation/approaches) for [Redgate’s SQL Change Automation](https://www.red-gate.com/products/sql-development/sql-change-automation/).  I picked this tool because it is easy to set up, integrates with SSMS, and I already had a demo setup.  I’m also [biased](https://www.red-gate.com/hub/events/friends-of-rg/friend/BobWalker) towards Redgate’s tooling.

By the end of this article, you will have a working proof of concept you can demo.

!toc

## Prep work

For this demo, you need a running SQL Server instance, an Octopus Deploy instance, and a CI server.  I recommend using a dev environment or your local machine for this proof of concept.  

## Required tools

You will need the following tools. The examples given use TeamCity and VSTS/TFS, but even if you’re using a different tool, the core concepts and UI for all the CI tools are very similar.

- Octopus Deploy:
    - [Free license for Octopus Cloud or Octopus Server](https://octopus.com/free)
- Redgate SQL Tool-belt
    - [14-day free trial](https://www.red-gate.com/dynamic/products/sql-development/sql-toolbelt/download).  
- Build Server/Continuous Integration (CI) tool (pick one):
    - [Jenkins](https://jenkins.io/download).
    - [TeamCity](https://www.jetbrains.com/teamcity/download/).
    - [TFS](https://visualstudio.microsoft.com/tfs/).
    - [VSTS](https://visualstudio.microsoft.com/team-services/).
    - [Bamboo](https://www.atlassian.com/software/bamboo/download).
- SQL Server Management Studio (SSMS):
    - [Free download](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms).
- SQL Server:
    - [SQL Express](https://www.microsoft.com/en-us/sql-server/sql-server-editions-express).
    - [SQL Developer](https://www.microsoft.com/en-us/sql-server/sql-server-downloads).

## Installing the software

If you run into issues installing any of these tools, please go to the vendor’s website for help.  If you need any help installing Octopus Deploy, please start with our [docs](https://octopus.com/docs/installation) or contact [support](mailto:support@octopus.com).

### Developer workstation

This is the machine you will use to make the schema changes and check them into source control.  When you install Redgate’s SQL Tool-belt, you will be prompt to install quite a bit of software.  You only need to install the following:
- SQL Source Control.
- SQL Prompt (it isn’t required, but it makes things much easier).
- SSMS Integration Pack.

### Build server/Continuous Integration tool

Both Octopus Deploy and Redgate have plug-ins for the major build servers/continuous integration tools:

- Jenkins:
    - [Octopus download](https://plugins.jenkins.io/octopusdeploy). 
    - [Redgate download](https://plugins.jenkins.io/redgate-sql-ci).
- TeamCity:
    - [Octopus download](https://plugins.jetbrains.com/plugin/9038-octopus-deploy-integration).
    - [Redgate download](https://www.red-gate.com/dlmas/TeamCity-download).
- VSTS/TFS:
    - [Octopus download](https://marketplace.visualstudio.com/items?itemName=octopusdeploy.octopus-deploy-build-release-tasks).
    - [Redgate download](https://marketplace.visualstudio.com/items?itemName=redgatesoftware.redgateDlmAutomationBuild).
- Bamboo:
    - [Octopus download](https://marketplace.atlassian.com/apps/1217235/octopus-deploy-bamboo-add-on?hosting=server&tab=overview).
    - [Redgate download](https://marketplace.atlassian.com/apps/1213347/redgate-dlm-automation-for-bamboo?hosting=server&tab=overview).

### Deployment target or database worker
Installing an Octopus Tentacle on SQL Server is a big no-no.  Our [documentation](https://octopus.com/docs/deployment-examples/sql-server-databases#SQLServerdatabases-Tentacles) goes into further details why.  

The preferred solution is to configure a jump box that sits between Octopus Deploy and SQL Server. Octopus supports two options for this:
* Deployment target
* Database worker

In this post, I'll add a deployment target but I wanted to mention workers as another good option. They're particularly useful for teams doing a lot of database deployments.

:::hint
Workers enable you to move deployment work onto other machines running in pools and database deployments are a common usecase for this. You can create a pool of dedicated workers that can be utilized for database deployments by multiple projects and teams. 

See [our documentation](https://octopus.com/docs/infrastructure/workers) for more information.
:::

For security purposes, I recommend running the Tentacle/Worker as a specific user account.  This way you can make use of integrated security. You can configure [Active Directory](https://octopus.com/docs/infrastructure/windows-targets/running-tentacle-under-a-specific-user-account) or use SQL Users instead.

For the jump box, you need to install the following items:
- SQL Change Automation PowerShell 3.0.
- SQL Change Automation.

## Sample project

For this walk-through, I modified the RandomQuotes project. The source code for this sample can be found in this [GitHub repo](https://github.com/OctopusDeploy/AutomatedDatabaseDeploymentsSamples).  Fork the repository so you can make modifications as you follow this article.

## Configuring the CI/CD pipeline

Everything you need is already checked into source control.  All we need to do is build it and push it to the SQL Server.  

### Octopus Deploy configuration

You need the step templates from Redgate to [create a database release](http://library.octopus.com/step-templates/c20b70dc-69aa-42a1-85db-6d37341b63e3/actiontemplate-redgate-create-database-release) and [deploy a database release](http://library.octopus.com/step-templates/7d18aeb8-5e69-4c91-aca4-0d71022944e8/actiontemplate-redgate-deploy-from-database-release).  When you browse the step template you might notice the step template to [deploy directly from a package](http://library.octopus.com/step-templates/19f750fb-2ce8-4361-859e-2dfcdf08a952/actiontemplate-redgate-deploy-from-package).  The state-based functionality for SQL Change Automation works by comparing the state of the database stored in the NuGet package with the destination database.  Each time it runs, it creates a new set of delta scripts to apply. The recommended process is:

1. Download the database package to the jump box.
2. Create the delta script by comparing the package on the jump box with the database on the SQL Server.
3. Review the delta script (this can be skipped in dev).
4. Run the script on SQL Server using the Tentacle on the jump box.

Using the step template to deploy from the package prevents the ability to review the scripts.  

This is the process I have put together for deploying databases.

![](octopus-database-deployment-overview.png)

This process does the following:

- The main SQL user for the database.
- The database.
- Adds the SQL user to the database.
- Adds the user to the role.  

If you want your process to do that, you can download those step templates from the [Octopus Community Step Template Library](http://library.octopus.com/listing).  

If this is the beginning of your automated database deployment journey, you don’t have to add all that functionality. The main steps you need from the above screenshot are:

![](octopus-database-required-steps.png)

Let’s walk through each one.  The download a package step is very straightforward, no custom settings aside from picking the package name:

![](octopus-database-download-package.png)

The Redgate - Create Database Release step is a little more interesting. The **Export Path** is the location the delta script will be exported to.  This must be a directory outside of the Octopus Deploy Tentacle folder because the **Redgate - Deploy from Database Release** step needs access to that path, and the Tentacle folder will be different for each step:

![](octopus-create-database-release-step.png)

I like to use a project variable:

![](octopus-database-project-variables.png)

The full value of the variable is:

```
    C:\RedGate\#{Octopus.Project.Name}\#{Octopus.Release.Number}\Database\Export
```

Other recommendations on this screen:
- I have supplied the username and password.  I recommend using integrated security and having the Tentacle running as a specific service account.  I don’t have Active Directory configured on my test machine, so I used SQL Users for this demo.
- Take a look at the [default SQL Compare Options](https://documentation.red-gate.com/sr1/using-sql-compare-options-in-sql-release/default-sql-compare-options-used-by-sql-release) and make sure they match what you need.  If they don’t, you need to supply the ones you want in the `SQL Compare Options (optional)` variable.  You can view the documentation [here](https://documentation.red-gate.com/sc11/using-the-command-line/options-used-in-the-command-line).  If you do decide to use custom options, I recommend creating a variable in a [library variable set](https://octopus.com/docs/deployment-process/variables/library-variable-sets), so those options can be shared across many projects.
- Use a custom filter in the event you want to limit what the deployment process can change.  I wrote a lengthy blog post about how to do that [here](https://www.codeaperture.io/2016/10/13/using-sql-source-control-to-filter-out-unwanted-items/).  My personal preference is to filter out all users and let the DBAs manage them.  Even better, let Octopus manage them since it can handle environmental differences.

The next step is approving the database release. I recommend creating a custom team to be responsible for this, but I prefer to skip this step in dev and QA:

![](octopus-approve-database-changes.png)

The create database release step makes use of the artifact functionality built into Octopus Deploy. This allows the approver to download the files and review them:

![](octopus-database-artifacts.png)

The final step is deploying the database release.  This step takes the delta script in the export data path and runs it on the target server, which is why I recommend putting the export path in a variable:

![](octopus-deploy-database-release.png)

That’s it for the Octopus Deploy configuration. Now it’s time to move on to the build server.

### Build server configuration

For this blog post, I use VSTS/TFS and TeamCity.  At a minimum, the build should do the following:

1. Build a NuGet package containing the database state using the Redgate plug-in.
2. Push that package to Octopus Deploy using the Octopus Deploy plug-in.
3. Create a release for the package, which was just pushed using the Octopus Deploy plug-in.
4. Deploy that release using the Octopus Deploy plug-in.

#### VSTS / TFS build

There are only three steps in VSTS/TFS to build and deploy a database:

![](vsts-build-database-overview.png)

The first step will build the database package from source control.  The highlighted items are the ones you need to change.  The subfolder path variable is relative.  I am using a sample Git repo which is why the _RedgateSqlChangeAutomationStateBased_ folder is in the path:

![](vsts-build-database-package.png)

The push package to Octopus step requires you to know the full path to the artifact generated by the previous step.  I’m not 100% sure how you would know without trial and error:

![](vsts-push-database-package.png)

Here is the full value, if you’d like to copy it:

```
    $(Build.Repository.Localpath)\RandomQuotes-SQLChangeAutomation.1.0.$(Build.BuildNumber).nupkg
```

The Octopus Deploy server must be configured in VSTS/TFS.  You can see how to do that in our [documentation](https://octopus.com/docs/packaging-applications/build-servers/tfs-azure-devops/using-octopus-extension).

The last step is to create a release and deploy it to dev.  After connecting VSTS/TFS with Octopus Deploy, you can read all the project names.  You can also configure this step to deploy the release to dev.  Clicking the **Show Deployment Progress** will stop the build and force it to wait for Octopus to complete:

![](vsts-create-octopus-database-release.png)

#### TeamCity

The TeamCity setup is very similar to the VSTS/TFS setup.  Only three steps are needed:

![](teamcity-build-sql-automation-overview.png)

The first step is the build database package step, and it has similar options to VSTS/TFS.  You need to enter the folder as well as the name of the package:

![](teamcity-redgate-build-database.png)

You have to enter a package version in the advanced options, or you will get errors from the Redgate tooling about an invalid package version:

![](teamcity-redgate-build-advanced-options.png)

The publish package step requires all three of the options to be populated.  By default, the Redgate tool will create the NuGet package in the root working directory:

![](teamcity-publish-package.png)

The final step is creating and deploying the release. Provide the name of the project, the release number, and the environment you are deploying to:

![](teamcity-create-database-release.png)

## See the CI/CD pipeline in action
Now it’s time to see all of this in action.  For this demo, I create a new database, *RandomQuotes_BlogPost_Dev*:

![](octopus-first-database-release-variables.png)

As you can see, I do not have any databases with that name. I have used this SQL Server as my test bench for automated deployments:

![](ssms-before-deployment.png)

Let’s take a quick look at the tables stored in source control:

![](github-tables-to-be-created.png)

If we open one of those files, we can see the create script generated by Redgate’s SQL Source Control:

![](github-database-quote-table.png)

Kick off a build, and let’s see that whole pipeline in action.  The build looks successful:

![](vsts-build-first-time.png)

No surprise, the deployment was successful in Octopus Deploy.  The VSTS/TFS build was set to wait on Octopus Deploy to finish deploying the database.  If the deployment failed the build would’ve failed:

![](octopus-first-database-release.png)

Going back to SSMS and we can now see the database and the tables have been created:

![](database-successful-deployment-ssms.png)

## Changing the database schema

That works with an existing project, but let’s make a small database schema change and test the process. There is a bit more setup involved with this:

1. Clone your forked repo to your local machine.
2. Open up SSMS and create a random quotes database on your local machine or dev.
3. In SSMS bind the source controlled database to the newly created database.  You can read how to do that in the [documentation](https://www.red-gate.com/products/sql-development/sql-source-control/resources/how-to-set-up-sql-source-control).

When linking the database to source control, you need to provide the full path to the folder where the source control is stored.  I store all my code in a folder called C:\Code.git. The full path is:

```
C:\Code.git\AutomatedDatabaseDeploymentsSamples\RedGateSqlChangeAutomationStateBased\db\src\
```

![](ssms-redgate-successful-link.png)

Now we can make the change to the database.  For this test, let’s just add in a stored procedure which will return a value:

![](ssms-sample-sproc.png)

Now we can commit that change to source control:

![](ssms-sample-commit.png)

Assuming the CI/CD pipeline is set to fire on commit, you should see that new sproc appear in dev.

## Conclusion

Database deployment automation does require a bit of prep-work, but the payoff is well worth the effort.  The auditing alone is worth it.  With this tool, I can now see who made a change, when a change was made, and when that change went into production.  In the past, that was kept in another location with a 50/50 shot of it being updated.

As you start on this journey, my recommendation is to add the manual verification step to all environments until trust has been established.  This will ensure you don’t accidentally check in a change that blows away half the team’s database changes.  

Until next time, happy deployments!

---

!include <database-deployment-automation-posts>
