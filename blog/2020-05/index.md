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

## Article scope

This article could talk about the entire database deployment process.  But that has already been done.  I've outlined an ideal database development process in an [earlier article](https://octopus.com/blog/designing-db-deployment-process).  [A follow-up article](https://octopus.com/blog/use-case-for-designing-db-deployment-process#draft-of-the-ideal-process) went into further deail.  

This article is going to focus on how to build trust in the deployment pipeline using database tooling with Octopus Deploy.  This article will work under the following assumptions:

1. Developers are using isolated or local databases for development work.
2. All changes are checked into a branch and are merged using a pull request or some sort approval process.
3. The build server has already:
    - Perform the necessary steps to ensure the database changes are syntactically correct.  
    - Any static analysis tools, such as SQL Enlight, are ran to find potential code smells.
    - Database unit tests (yes they exist!) and if possible, integration tests are run.
    - Packaged up the database changes into a zip file or NuGet package and pushed to Octopus Deploy.

## Assume zero trust on day one

When designing and implementing an automated database deployment process I start with this mindset:

:::highlight
I need to prove to myself and everyone on my team a specific deployment will **_NOT_** result in data loss or a production outage.    
:::

Since the start of manned space flight in 1960s NASA has done a [launch status check](https://en.wikipedia.org/wiki/Launch_status_check) prior to launch.  The key people in the room tell the flight controller they are ready (go) or not ready (no-go) for a launch.  I am going to leverage the following features in Octopus Deploy to provide something similar for the automated database deployment.

- [Manual Interventions](https://octopus.com/docs/deployment-process/steps/manual-intervention-and-approvals) which will pause the deployment and wait for someone to approve or reject the change.
- [Artifacts](https://octopus.com/docs/deployment-process/artifacts) to store the delta scripts for the DBAs (or whomever) to review during the manual interventions.
- [Run Conditions](https://octopus.com/docs/deployment-process/conditions#run-condition) to always notify the development team of the deployment outcome and to page the DBAs on production failures.
- [Custom Log Levels](https://octopus.com/docs/deployment-examples/custom-scripts/logging-messages-in-scripts) used by the run a script step to notify DBAs of potential problems in the scripts.
- [Email and other notification options](https://octopus.com/docs/deployment-process/steps/email-notifications) to let the approvers know of a pending change.
- [Guided failure mode](https://octopus.com/docs/managing-releases/guided-failures) which will pause the deployment on failure and let people look into it and potentially try again.
- [Output variables](https://octopus.com/docs/projects/variables/output-variables) to enable skipping unnecessary steps in the deployment.  For example, if the tooling reports no database changes, there is no point in running the deploy database change step.
- [Audit Log](https://octopus.com/docs/administration/managing-users-and-teams/auditing) to know who kicked off the deployment, who approved, and when it all happened.
- [Users and Teams](https://octopus.com/docs/administration/managing-users-and-teams) to only allow DBAs permission to kick off Production deployments, but still allow developers permissions to deploy to lower environments.

## Bare Bones Deployment Process

If we had total trust in the process, we would only need the deployment step.  Let's start there.

:::highlight
This article uses [DbUp](https://dbup.github.io/), a cross-platform database deployment tool, to handle the database deployments.  The core concepts of this article can be used for pretty much any database deployment tool to any database server.
:::

![](deployment-process-start.png)

Right now, this process will take the scripts in the package and run them on the database.  It'd be nice to know what those scripts are.  Let's add a step to generate a delta script or report.

![](adding-delta-report.png)

The `Create Delta Report` step has this line at the end of the script which uploads the script to Octopus Deploy as an artifact.

```PowerShell
New-OctopusArtifact -Path "$generatedReport" -Name "$environmentName.UpgradeReport.html"
```

:::highlight
The method for creating a delta report will vary based on your database deployment tooling.  Please consult the tooling's documentation to find out how.  In some cases, our [docs](https://octopus.com/docs/deployment-examples/database-deployments) have examples which you can leverage. 
:::

During a deployment, that artifact will appear in two places on the screen.  

![](octopus-deploy-artifact.png)

Clicking on either of them will download the artifact.  

![](downloaded-artifact.png)

The deployment screen also shows who started the deployment and when it was started.  Both this audit log and artifact will be kept until the end of time.  Unless you have configured [retention policies](https://octopus.com/docs/administration/retention-policies).  Which, if so, the audit log and artifact will be kept until the deployment is deleted by the retention policy.

![](audit-deployment-log.png)

So far so good.  We now have the following:

1. A delta report which anyone can download and see what scripts were ran.
2. An audit history of who started the deployment and when it was started.

## Approvals 

But what if the delta report shows a script with a command like "drop table" or "drop column?"  Is that okay?  Maybe, maybe not.  It doesn't make sense to reject that script outright.  It makes more sense to pause the deployment, review the scripts, and if something doesn't look right, find the person who created the script and discuss it with them.  To do that, we will add a manual intervention.  

An interesting thought comes up when creating the manual intervention.  A team must be selected.  I know what you're thinking.  "The team should be the DBAs!"  This step is going to run for all environments.  When _should_ a DBA review changes?  They are busy keeping the databases servers up and running.  Having to review _every_ delta script for _every_ deployment for _every_ would be a full time job.  

Putting two teams, developers and DBAs won't work.  That means a developer **OR** a DBA could approve the deployment.  While acceptable for `Development` or `Test`, it is not acceptable for `Production`.

![](creating-manual-intervention.png)

In this example, it makes sense for DBAs to approve the delta script for `Staging` and `Production`.  Meanwhile, the developers can approve the delta script for `Development` and `Test`.

:::highlight
I like to add the Octopus Managers team to all manual interventions.  This way they can take responsibility in the event of an emergency and all the DBAs are off-site at a all-day meeting.  They are there for a "break glass in case of emergency."  
:::

![](dba-approve-delta-script.png)

The resulting deployment process will have two manual intervention steps.

![](deployment-process-with-manual-interventions.png)

In my experience, the ability to pause and review the delta script before it is ran goes a long way in building trust.  If the approver sees something they disagree with they can abort the deployment.  Even better, the person who took responsibility for the approval is in the audit log.  We can see I started the deployment as well as approved it (which outside of a demo / test environment is a big no-no).

![](manual-interventions-during-deployment.png)

## Notifications

Chances are DBAs are not watching Octopus Deploy all day, every day.  They need to be notified about approvals they are responsible for.  But what should they do after they approve a deployment?  Watch it finish?  That could take quite a while.  That doesn't seem like a productive use of their time.  And if the deployment is successful, they probably don't need to do anything else.  Only if the deployment fails do they need to step in.

For this example, I am going to include the email notifications directly in the process.  You can also leverage the [subscription](https://octopus.com/docs/administration/managing-infrastructure/subscriptions) feature in Octopus Deploy.

The initial fields on the email form are fairly straight forward.  The subject and body is what caused me to pause.  The DBAs might be getting dozens of these emails a day.  It should be very easy for them to approve the deployment.  The subject line should include details about the deployment.  If something doesn't look right they can forward you the email and provide their feedback.  The body should include a deep link for the approver to click on.

![](notification-subject-and-body.png)

The subject is getting cut off in the screenshot.  The subject is:

```
Pending approval for #{Octopus.Project.Name} #{Octopus.Release.Number} to #{Octopus.Environment.Name}
```

The failure notification is a bit different.  The priority of the email should change from normal to high.  And the run condition should be configured to only run the step on failure.

![](failure-notification.png)

:::highlight
This example is using email notifications, but you can also leverage other tools such as Slack, Teams, VictorOps, and others.
:::

Finally, the team responsible for the application should always be notified of the status of the deployment.  The deployment team uses Slack, so we will drop a similar message in their slack channel.  This example is using the [Slack - Send Simple Notification](https://library.octopus.com/step-templates/99e6f203-3061-4018-9e34-4a3a9c3c3179/actiontemplate-slack-send-simple-notification) step template from the library.

![](deployment-process-with-notifications.png)

With these notifications in place, the key people in this process are now kept up-to-date with what is going on with the application.

## Helping the approvers

As time goes on, the number of deployments the DBAs have to approve will exponentially grow.  It is the nature of automation.  When teams feel comfortable with the tooling and trust the process they will naturally do more deployments.  

More deployments is a double-edge sword, the downside is it results in less time is available to review database changes.  A DBA or developer could miss a schema change that causes significant damage.  The process should help them.  Thankfully, Octopus Deploy is an API first application.  Using the API it is possible download the delta report or sql artifacts to find any potentially damaging statements.  

```PowerShell
$CommandsToLookFor = "Drop Table,Drop Column,Alter Table,Create Table,Create View"

$OctopusURL = ## YOUR URL
$APIKey = ## YOUR API KEY
$FileExtension = "SQL"
$SpaceId = $OctopusParameters["Octopus.Space.Id"]
$DeploymentId = $OctopusParameters["Octopus.Deployment.Id"]

Write-Host "Commands to look for: $CommandsToLookFor"
Write-Host "Octopus Url: $OctopusUrl"
Write-Host "SpaceId: $SpaceId"
Write-Host "DeploymentId: $DeploymentId"
Write-Host "File Extension: $FileExtension"

$header = @{ "X-Octopus-ApiKey" = $APIKey }

[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
$artifactUrl = "$OctopusUrl/api/$SpaceId/artifacts?take=2147483647&regarding=$DeploymentId&order=asc"
Write-Host "Getting the artifacts from $artifactUrl"
$artifactResponse = Invoke-RestMethod $artifactUrl -Headers $header
$fileListToCheck = @()

foreach ($artifact in $artifactResponse.Items)
{
    $fileName = $artifact.Filename

    if ($fileName.EndsWith($FileExtension))
    {    	
        Write-Host "The artifact is a SQL Script, downloading"
        $artifactId = $artifact.Id
        $artifactContentUrl = "$OctopusUrl/api/$SpaceId/artifacts/$artifactId/content"
        Write-Host "Pulling the content from $artifactContentUrl"
        $fileContent += Invoke-RestMethod $artifactContentUrl -Headers $header
        Write-Host "Finished downloading the file $fileName"
        
        $sqlFileToCheck = @{
        	FileName = $fileName;
            Content = $fileContent;
        }
        
        $fileListToCheck += $sqlFileToCheck                
    }    
}

if ($fileListToCheck.Length -le 0)
{
	Write-Highlight "No sql files were found"
    
    Exit 0
}

Write-Host "Looping through all commands"
$commandsToCheck = $CommandsToLookFor -split ","
foreach ($sqlFile in $fileListToCheck)
{	    
	foreach ($command in $commandsToCheck)
    {
    	Write-Host "Checking $($sqlFile.FileName) for command $command"
    	$foundCommand = $sqlFile.Content -match "$command"
    
    	if ($foundCommand)
        {
        	Write-Highlight "$($sqlFile.FileName) has the command '$command'"            
        }
    }
} 
```

When certain commands are found the script will leverage the `Write-Highlight` [logging command](https://octopus.com/docs/deployment-examples/custom-scripts/logging-messages-in-scripts) provided by Octopus Deploy.  The message will appear in the deployment log.  

![](write-highlight-in-deployment.png)

Seeing those messages should hopefully make it a little easier on the approver.  

## Handling errors and failures

Errors and failures will happen.  A number of them occur because of something out of the control of Octopus Deploy.  A network failure, a SQL Server restart, incorrect permissions, and missing accounts are amongst the most common.  Once the issue is fixed, it would be much better just to try the failed step again.  No sense in bugging the DBA to review the same script again when a permission issue caused a failure.  

This is the scenario [guided failure mode](https://octopus.com/docs/managing-releases/guided-failures) was designed for.  It can be enabled at the per project or per environment level.  For this example, it is enabled at the project level.

![](use-guided-failure-mode.png)

Just like a manual intervention, the deployment will pause when a failure is detected.  Octopus Deploy provides the option to `Fail`, `Ignore` or `Retry` the step.  

![](guided-failure-during-deployment.png)

This will give the users the chance to gracefully handle a failure.

## Separation of concerns

Separation of concerns is a common practice.  The person who made the code change shouldn't be the person who approved it or deployed it to `Production`.  But developers can deploy to `Development` and `Test`.  In Octopus Deploy, it is possible to configure the teams to do this.

There are five roles which help achieve this in Octopus Deploy.

- Deployment Creator - People assigned to this role can kick off deployments.  In this example, DBAs will be assigned this role for `Production`.
- Project Contributor - People assigned to this role can edit the project, but cannot create or deploy releases.  In this example, developers will be assigned this role for `Production`.
- Project Deployer - People assigned to this role can do everything a project contributor can do, including deploying a release.  In this example, developers will be assigned this role for the lower environments.
- Project Viewer - People assigned to this role have a read-only view of the project.  Perfect for QA, Business Owners and other people who don't have permissions to change a process but still want ot see the status.

![](built-in-user-roles.png)

An interesting question is raised in looking at this example.  The DBAs approve the deployment via the manual intervention in `Staging`.  Should they be the ones who trigger the deployment?  That doesn't make a lot of sense.  The developers should trigger the deployment to `Staging`.  

The developer team user roles will be:

![](developer-team-permissions.png)

While the DBA's permissions would be:

![](dba-permissions.png)

## Adjusting the DBAs' role in the process

With those permissions, the DBAs' role in this process is still a bit odd.  They are the ones who will also trigger a deployment to `Production`.  Should they also have to manually approve the deployment as well?  At first blush, it makes sense.  They should approve a `Production` release.  

Here is the real question: how often has a development team changed something during a `Production` deployment because a DBA found something they didn't like?  

The answer is probably never.  When an outage window has been communicated to users and customers, the `Production` deployment is too late for a DBA to voice their concerns.  Unless the DBA can say with 100% certainty a problem is going to happen, the production deployment will proceed.

A `Production` deployment is too late for a DBA to be involved.  A much better approach is for a delta script for `Production` to be created during the `Staging` deployment.  That way the DBA can approve both of them at the same time.  

![](earlier-dba-approval.png)

When the DBA creates the `Production` deployment, they can also review the delta report generated during the `Staging` deployment.  The added bonus is the DBA can now schedule the deployment and they don't have to be online.

![](scheduled-deployment.png)

That being said, don't expect the DBA to schedule a deployment and not be online when you start out.  It will take quite a number of deployments before they will trust the team, trust the process, and trust themselves to understand the tool.

## Future Iterations

All the manual approvals and notifications will create noise.  When starting out, that noise is good.  The team is still learning the tooling.  Eventually, the team will become comfortable with the tooling.  Rather than help the team, the manual approvals and notifications will annoy and eventually hinder the team.  

It isn't wise to remove those notifications and approvals.  Something damaging could slip through the pull request process.  The process has a step which automatically finds potential problems, `Check SQL Artifacts for Schema Change Commands`.  That script can be modified to set [output variables](https://octopus.com/docs/projects/variables/output-variables).  If a potentially damaging command is found then the deployment must go through the approval process.

```PowerShell
$ApprovalRequired = $false
Write-Host "Looping through all commands"
$commandListToCheck = $CommandsToLookFor -split ","
foreach ($sqlFile in $fileListToCheck)
{	    
	foreach ($command in $commandListToCheck)
    {
    	Write-Host "Checking $($sqlFile.FileName) for command $command"
    	$foundCommand = $sqlFile.Content -match "$command"
    
    	if ($foundCommand)
        {
        	Write-Highlight "$($sqlFile.FileName) has the command '$command'"
            $ApprovalRequired = $true
        }
    }
} 

if ($approvalRequired -eq $false)
{
	Write-Highlight "All scripts look good"
}
else
{
	Write-Highlight "One of the specific commands we look for has been found"
}

Set-OctopusVariable -name "DBAApprovalRequired" -value $ApprovalRequired
```

Output variables are a bit verbose.  To make them easier to use, set another variable to the result.

![](output-variable-reference.png)

In the notification and manual intervention steps, the variable run condition can be set to be:

```
#{if Octopus.Deployment.Error == ""}#{Project.DBA.ApprovalRequired}#{/if}
```

![](variable-run-condition-set.png)

## Preventing bad actors

One flaw in this process is it assumes developers don't go into the deployment process and disable or delete the manual intervention steps.  Trust, but verify is our recommended approach.  

For starters, we are working on a process as code which integrates with Git.  If it integrates with Git, it can then go through a pull request process.  

[This video](https://www.youtube.com/watch?v=i4qYdsYyu9s&list=PLAGskdGvlaw3-cd9rPiwhwfUo7kDGnOBh&index=19&t=0s) also outlines a process to automatically audit your projects.

Finally, you can configure [subscriptions](https://octopus.com/docs/administration/managing-infrastructure/subscriptions) to notify admins when changes are made.

## Conclusion

Without trust, automated database deployments will fail.  The process created in this article does the following to help build that trust.  

1. Generates a delta report and publishes it as an Octopus Artifact for anyone to review.
2. Keeps track of who triggered the deployment, who approved the deployment and when everything happened.
3. A script will download the delta report and look for potential problem commands.
4. The deployment will pause in each environment, except `Production`, for approvers to review and approve the delta report.
5. Notifications are sent to key people at various times during the process.
6. Developers have permissions to deploy to `Development`, `Test`, and `Staging`, but DBAs must approve the deployment to `Staging`.
7. DBAs approve and review changes for both `Staging` and `Production` during a `Staging` deployment.  This gives them much more time to review the changes and raise any concerns prior to a `Production` push.  

I am not naive enough to believe this will solve all the potential trust concerns.  The process isn't perfect, for example, it doesn't integrate with a Production Change Request system and it doesn't integrate with issue trackers.  While important, I felt those items muddied the waters for this article.  My goal for this article was to provide tips and techniques for you to feel confident to start your own POC or Pilot for automated database deployments.  

The deployment process from this article can be found on our [samples instance](https://samples.octopus.app/app#/Spaces-106/projects/dbup-sql-server/deployments).  

Until next time, happy deployments!