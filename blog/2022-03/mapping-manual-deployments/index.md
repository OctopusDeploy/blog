---
title: Mapping manual deployments with Octopus Deploy
description: Find out how to map your manual deployments to create a template that helps you start your automation journey.
author: steve.fenton@octopus.com
visibility: private
published: 9999-01-01-0000
metaImage: 
bannerImage: 
bannerImageAlt: 
isFeatured: false
tags: 
  - DevOps
  - Getting Started
  - Product
---

While Octopus Deploy makes complex deployment automation easy, you might have a scenario where you are still running a manual deployment. We have talked to customers with a legacy application that needs some special steps during the release process, or who have esoteric technology that makes it hard to work out where to start the automation journey.

In this post, you'll find out how you can model your manual deployments using manual intervention steps and the benefits this will bring to your release process.

If you are new to Octopus, you might find this a useful way to advance from your current deployment process to full automation. Existing customers may find that this technique allows them to bring to their manual releases all the benefits of approvals and tracking that they get with automated deployments.

## Benefits of using Octopus for manual deployments

- Use lifecycles to track deployments across multiple environments
- Ensure each step is completed by an authorised team member
- Improve the reliability and traceability of deployments
- Satisfy audit requirements with less effort

## Creating a checklist

When you have a manual release process, there is usually a document or checklist that explains the steps required to install the software along with who can perform each step. Before you re-create this in Octopus, it is worth spending some time refining the stages to agree who does what, in which order.

If your document is lengthy, you might find you can divide it with headings that will give you a natural task list.

You should end up with something like the following checklist:

| Step   | Title                                                        | Who  |
|--------|--------------------------------------------------------------|------|
| 1      | Backup the database                                          | DBA  |
| 2      | Upgrade the database                                         | DBA  |
| 3      | Copy the new application to a temporary folder on the server | Ops  |
| 4      | Delete the configuration file in the temporary folder        | Ops  |
| 5      | Copy the live configuration file into the temporary folder   | Ops  |
| 6      | Add any new settings to the configuration file               | Ops  |
| 7      | Copy the temporary folder into the live folder               | Ops  |
| 8      | Check the application loads as expected                      | Test |

If you didn't have a checklist before you started this exercise, you are likely to find that the checklist already increases the reliability of your deployments by ensuring the steps all occur and are done in the right order. Now we can transfer this into Octopus to get the benefits of workflow management and tracking.

## Create teams

The checklist contains three teams who are responsible for the deployment: DBAs, Ops, and Test. When you create the process in Octopus, each step will have a *responsible team*. Follow these steps to add each team to Octopus:

- Navigate to **{{ Configuration,Teams }}**
- Select **ADD TEAM**
- Enter the **New team name**, for example "DBA Team"
- Select **SAVE**
- Open the **USER ROLES** tab
- Select **INCLUDE USER ROLE**
- Choose **Project Deployer** from the list
- Select **DEFINE SCOPE**
- Under **Select project groups** choose "Default Project Group"
- Select **APPLY**
- Finally, select **SAVE**

:::hint
The **Project deployer** role grants the user all project contributor permissions, plus: deploying releases and executing runbooks.
:::

You can add team members by selecting the **ADD MEMBER** option. It is possible to assign specific users or roles to a team.

With the three teams set up with their respective team members, you are ready to model the checklist as a process.

![The teams list showing the DBA team, the Ops team, and the test team](manual-deployment-teams.jpg)

## Create a lifecycle

The lifecycle for your manual deployment might not match a lifecyle you are using for automated deployments. If this is the case, you can create a new lifecyle to use with your manual release process.

 - Navigate to **{{ Infrastructure,Environments }}**
 - Select **ADD ENVIRONMENT**
 - Enter a **New environment name**, for example "Manual Test Environment"
 - Select **SAVE**

:::hint
Normally after adding an environment, you would add deployment targets. As you are modelling a manual deployment you can leave the environments empty at this stage.
:::

Once you have created your environments, you can define a lifecyle that sets the order for releases. For example, you can enforce that a release must be deployed to the test environment before it is promoted to the live environment.

- Navigate to **{{ Library,Lifecycles }}**
- Select **ADD LIFECYCLE**
- Enter the **Name** of the lifecyle, for example "Manual Deployment Lifecycle"
- Under **Phases** select **ADD PHASE** to add each environment
- Select **SAVE** when you have finished adding phases

Your completed lifecyle should contain a phase for each environment as shown below.

![A manual deployment lifecycle with a test phase and a live phase](manual-lifecycles.jpg)

## Modelling the steps

To add your checklist items as steps, you will need to create a new project. You can do this by navigating to **Projects** and selecting **ADD PROJECT**.

- Add a **New project name**, such as "Manual Deployment"
- Select the **Project group** where you want to add the new project, for example "Default Project Group"
- Select the **Lifecycle**, for example the "Manual Deployment Lifecycle" you created previously
- Select **SAVE**

You will now see your project listed under the **Default Project Group**. You can also see the project listed on your dashboard.

You now have all the resources ready to add the steps, so navigate to **{{ Projects,Manual Deployment,Process }}** to begin.

You will add a step for each of the checklist items to create a process that will drive your manual deployments. So, repeat the following process for each of the items in your list.

 - Click **ADD STEP**
 - Filter the step templates using the search term "Manual Intervention"
 - Click **ADD** on the **Manual Intervention Required** step
 - Enter the title from your checklist into **Step name**, for example "Backup the database"
 - If you have documentation for this step, you can add it to the **Instructions** field, which uses markdown for formatting
 - Use the list under **Responsible teams** to select the correct team, for example "DBA Team"
 - Click **SAVE** to complete the step configuration

:::hint
You may want to adjust the default option for blocking deployments to prevent multiple concurrent releases. Under the **Block Deployments** setting, select **Prevent other deployments while awaiting intervention**.
:::

When you have added your steps navigate to **{{ Projects,Manual Deployment,Process }}** and review the process overview, which should look like the below example.

![The process shows the checklist steps and the manual deployment lifecycle](manual-deployment-process.jpg)

You can adjust the steps and the lifecycle at any time if you need to change them.

## Using the process to track a release

Although you don't have a package to deploy, you can still track the deployment using a release in Octopus.

- Navigate to **{{ Projects,Manual Deployment,Process }}**
- Select **CREATE RELEASE**
- Enter a **Version**, you may want to follow on from an existing version number you have published
- Select **SAVE**

You can now track the manual deployment to each environment and the release can only proceed to the *manual live environment* if it first gets deployed to the *manual test environment* as controlled by the lifecycle you configured.

- From the release screen for your new version, select **DEPLOY TO MANUAL TEST ENVIRONMENT**
- A confirmation screen will appear, review the information and select **DEPLOY**

The deployment is created for the release, and the first manual intervention step is ready to be picked up by a member of the DBA team. The release is given a manual intervention icon to show you that it is waiting for human intervention.

![The Octopus dashboard with an orange eye icon that indicates a manual intervention is required](manual-intervention-needed.jpg)

Only a member of the DBA Team can assign themselves the **Backup the database** task. They can do this by clicking on the release from the dashboard and selecting **ASSIGN TO ME**.

The process for each step as the manual deployment progresses is:

- Select **ASSIGN TO ME** on the step
- Use the step instructions on the task summary to complete the step
- Enter any notes, logs, or output from the manual step into the **Notes** field
- Select **PROCEED** to complete the step

As each step is completed, the next step will become available to members of the relevant team.

The task log records the date and time the step was done, who completed it, and any notes they entered.

```no-highlight
                    |   == Success: Step 1: Backup the database ==
15:13:26   Verbose  |     Backup the database completed
13:21:28   Verbose  |     Resuming after completion
13:21:28   Info     |     Submitted by: Sarah (DBA) at 2022-03-03T13:21:24.3250260+00:00
...

...
                    |   == Success: Step 8: Check the application loads as expected ==
13:25:53   Verbose  |     Check the application loads as expected completed
13:27:33   Verbose  |     Resuming after completion
13:27:33   Info     |     Submitted by: Tina (Test) at 2022-03-03T13:27:25.7892278+00:00
13:27:33   Info     |     Notes: 42 passed
                    |     0 failed
                    |     PASS
```

After your first deployment, you may decide to adjust the process.

## Look for opportuinities to automate

When you first started mapping your manual deployment process, automation may not have been your goal. However, you now have a starting point for automation and can review the steps to see if you can progress them through these levels of maturity:

1. Manual steps
1. A script that can be manually run
1. A scripted step that runs automatically

For example, the database backup is currently being done manually, but the instructions for this step could be updated to include a script that will run the backup. Although the script will still be manually run, it will make it more likely that the backup will be done the same way each time.

Once you have a script instruction that is being manually executed, you could move that script into a new step. The existing manual intervention step could become a "check" that the automation worked as expected and when you have confidence in the automation you could remove the manual step entirely.

You may find that there are existing step templates that will help you with your automation, for example there are step templates for backing up SQL databases on AWS, Azure, or SQL Server. You can find step templates by navigating to **{{ Library,Step Templates,BROWSE LIBRARY }}**.

Once you have introduced more automation, you should review how often you deploy. If your deployment was only released once a month because it took so long to roll it out, you could look at moving to weekly releases.

Frequent deployments are correlated to high-performance, not simply because releasing often is a high-performance trait, but beause of what you learn and adjust to achieve it.

## Summary

## Further reading

- Docs
- Community Slack
- Where else?





