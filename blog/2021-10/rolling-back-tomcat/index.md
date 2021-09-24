---
title: Rolling back a Tomcat deployment
description: Learn how to rollback an application deployed to a Apache Tomcat server.
author: shawn.sesna@octopus.com
visibility: private 
published: 2022-10-11
metaImage: 
bannerImage: 
bannerImageAlt: 
isFeatured: false
tags:
 - 
---

The DevOps feedback loop usually has automated processes in place to capture any issues as early in the pipeline as possible.  While these automated processes allow for early detection, bugs can still find their way into Production code.  Some of these bugs are severe enough to warrant backing out the recently deployed version.  The process of reverting changes is often referred to as a rollback.  In this post, I'll discuss some different rollback strategies when using an Apache Tomcat web server.

## Initial deployment process
For this post, you'll be deploying the PetClinic application to an Apache Tomcat web server.  The PetClinic application requires a database and uses Flyway to perform database migrations.  The example process should look something like this:

1. Create Database If Not exists
1. Deploy Database Changes
1. Deploy PetClinic Web App
1. Verify deployment
1. Notify Stakeholders

![](octopus-original-deployment-process.png)

This post assumes you're already familiar with the steps included in this deployment process and will not go into detail for each step.

## Deploying the previous version
Using Octopus Deploy, one method of rolling back is to redeploy the previous release of the application.  This is as easy as selecting the previous release from the **Releases** screen, then clicking the **REDEPLOY** button next to the desired environment.

![](octopus-redeploy-release.png)

All steps in the deployment process are executed as they were configured when the release was created.  For Apache Tomcat, this means that the package will be rextracted on the Tomcat server, variable substituion performed, and repackaged before being sent to the Tomcat Manager for deployment.  Depending on the size of the package, this could be a quick operation, or take quite some time.

## Simple rollback
As previously stated, all steps in the deployment process will be reexecuted as they did the first time.  While the built-in method of redeployment is effective, it doesn't automate activities such as marking the release as being bad.  Automation of marking a releas as bad first needs to know that a rollback is occuring.  This requires modification to the deployment process, the new process would look something like this:

1. Calculate Deployment Mode
1. Create Database If Not exists (skip during rollback)
1. Deploy Database Changes (skip during rollback)
1. Deploy PetClinic Web App
1. Verify deployment
1. Notify Stakeholders
1. Block Release Progression (only during rollback)

:::info
Using the `Notes` field of a deployment step, we can provide documentation for the steps and which mode they will run in.
:::

![](octopus-simple-rollback-process.png)

You'll notice the addition of two new steps (`Calculate Deployment Mode` and `Block Release Progression`) as well as some conditions applied to existing steps.

### Calculate Deployment Mode
The first part of a rollback process is to determine whether it is a deployment, a rollback, or a redeployment operation.  The Octopus Solutions team has developed the community step template [Caclulate Deployment Mode](https://library.octopus.com/step-templates/d166457a-1421-4731-b143-dd6766fb95d5/actiontemplate-calculate-deployment-mode) which not only figures out which mode the deployment is in, it also produces a number of [output variables](https://octopus.com/docs/projects/variables/output-variables) containing variable [run condition](https://octopus.com/docs/projects/steps/conditions#run-condition) syntax.

### Database steps
The `Create Database If Not Exists` and `Deploy Database Changes` steps do not need to be run in a rollback, so you'll want to configure those to be skipped.  The `RunOnDeploy` output variable from the `Calculate Deployment Mode` can be applied to the variable run condition of these steps to skip them

```
#{Octopus.Action[Calculate Deployment Mode].Output.RunOnDeploy}
```

### Block Release Progression
Blocking the progression of a release has been in Octopus Deploy for quite some time, however, needed to be done via the UI or an API call.  The Octopus Solutions team recognized that automating this activity is critical when dealing with rollbacks and developed the [Block Release Progression](https://library.octopus.com/step-templates/78a182b3-5369-4e13-9292-b7f991295ad1/actiontemplate-block-release-progression) template which automates the blocking of a release.  This step should only run during a rollback operation, use the following output variable from `Calculate Deployment Mode` as the variable run condition to ensure this:

```
#{Octopus.Action[Calculate Deployment Mode].Output.RunOnRollback}
```

## Complex rollback
Both the built-in method and Simple Rollback method will extract and repackage the `.war` file before delivering it to the Tomcat server for deployment.  If your application is large, this could potentially take quite a bit of time.  Using the Tomcat [Parallel Deployment](https://tomcat.apache.org/tomcat-7.0-doc/config/context.html#Parallel_deployment) feature, it's possible to perform a rollback in mere seconds.

### Tomcat parallel deployments
The feature of parallel deployments was first introduced in Tomcat version 7.  The parallel deployments feature allows you to deploy multiple versions of the same application to a Tomcat server.  Once a newer version of the application is in a running state, new sessions will run on the new version and existing sessions will continue to run on the older version until they expire.  This feature requires that you supply a version number with the context path.  The Tomcat server will combine the version number an context path and rename the deployed `.war` to `<contextpath>##<version>.war`

### Complex rollback process
To implement a rollback process using the parallel deployments feature, you'll need to modify your process to look something like this:

1. Calculate Deployment Mode
1. Rollback reason (only during rollback)
1. Create Database If Not exists (skip during rollback)
1. Deploy Database Changes (skip during rollback)
1. Stop App in Tomcat (run on rollback or run on deployment and rollback)
1. Deploy PetClinic Web App (only during deployment or redeployment)
1. Start App in Tomcat (only during rollback)
1. Verify deployment
1. Notify Stakeholders
1. Block Release Progression (only during rollback)

![](octopus-complex-rollback-process.png)

Let's go over the changes to the process to make this work.

#### Rollback reason
The `Rollack reason` step is a Manual Intervention that promptes the user for the reason for the rollback.  The reason specified is used for the `Reason` field in the `Block Release Progression` step further down in the process.  This step only runs during a rollback so it needs to have the variable run condition set to the following:

```
#{Octopus.Action[Calculate Deployment Mode].Output.RunOnRollback}
```

#### Stop App in Tomcat 
Both the `Stop` and `Start` steps use the `Start\Stop App in Tomcat` step.  The Stop step is optional when in `Deployment Mode`, but necessary in `Rollback Mode` as Tomcat will funnel new sessions to the most recent version of the deployed application.  When rolling back, we need to stop the version that is bad so that the previously deployed version will now get the sessions.  The variable run condition can be 

```
#{Octopus.Action[Calculate Deployment Mode].Output.RunOnRollback}
```
If you want the step to execute whenever it detects something has already been deployed to the environment and stop it, use
```
#{if Octopus.Release.CurrentForEnvironment.Number}True#{/if}
```

The version number for the `Advanced Options` for the Tomcat step is below with the selected action of `Stop`

```
#{Octopus.Release.CurrentForEnvironment.Number}
```

![](octopus-tomcat-stop-advanced.png)

#### Start App in Tomcat
For this post, I'm stopping the previously deployed application before starting the one for the release.  This means that in a rollback scenario, I need to start the version that I'm rolling back to.  The version number specified in the `Advanced Options` for the Tomcat step is below with the selection action of `Leave the application running`

```
#{Octopus.Release.Number}
```

![](octopus-tomcat-advanced.png)

This step needs to be configured to run in `Rollback Mode` only using the following variable run condition:

```
#{Octopus.Action[Calculate Deployment Mode].Output.RunOnRollback}
```

#### Deploy PetClinic Web App
This step needs the same `Advanced Options` set as the `Start App in Tomcat` step, supllying `#{Octopus.Release.Number}` for the version being deployed.  (See image in `Start App in Tomcat` for reference.)

#### Block Release Progression
In the Simple Rollback scenario, the `Reason` field for the Block Progression is being statically set.  The `Rollback reason` step prompts the user for the reason for the rollback, the `Notes` output variable can be used as the input of `Reason` to have a more meaningful message for the blocked release.  Modify the `Reason` field to use the `Notes` output varaiable from the `Rollback reason` step like this:

```
#{Octopus.Action[Rollback reason].Output.Manual.Notes}
```

:::warning
Enabling the [undeployOldVersions](https://tomcat.apache.org/tomcat-9.0-doc/config/host.html) feature in Tomcat has the potential of interfering with the complex rollback strategy.
:::

Parallel deployments will appear similar to this in the Tomcat Manager

![](tomcat-manager-multiple.png)

### Cleaning up old versions
The retention policies of Octopus Deploy help keep your targets clean, removing old versions that have been deployed.  However, the Tomcat Manager places the `.war` files in it's own folder which is unknown to Octopus Deploy.  When not using the parallel deployment feature, the `.war` is simply overwritten with the new version and the application is deployed.  The parallel deployments feature of Tomcat renames the `.war` to `<contextpath>##<version>.war` so they are unique.  These files will continue to accumulate unless an `undeploy` operation is executed against the versioned application entry.  To assist in Tomcat maintenance, the Octopus Deploy Solutions team developed [Undeploy Tomcat Application via Manager](https://library.octopus.com/step-templates/34f13b4c-64e1-42b4-ad1a-4599f25a850e/actiontemplate-undeploy-tomcat-application-via-manager), a community step template that will perform undeploy operations on a Tomcat server and is currently available in Bash syntax.

## Conclusion
In this post I walked through a few ways that you can rollback an application deployment to an Apache Tomcat web server.  I hope one of these methods will help you in your DevOps journey.  Happy deployments!