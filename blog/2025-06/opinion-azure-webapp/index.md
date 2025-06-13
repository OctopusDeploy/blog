---
title: Opinions on deploying an Azure Web App
description: This post discusses my opinion on how to deploy an Azure Web App.
author: shawn.sesna@octopus.com
visibility: public
published: 2026-01-01-1400
metaImage: blog-opinions-on-azure-web-app-2025-750x400.jpg
bannerImage: blog-opinions-on-azure-web-app-2025-750x400.jpg
bannerImageAlt: Illustration of people having a discussion with speech bubbles, with a central icon of the Azure App Service logo. A green checkmark suggests a verified or recommended practice.
isFeatured: false
tags: 
 - DevOps
 - AI
 - Azure
 - Deployment Patterns
---

Pushing code for an Azure Web App is not particularly difficult. Microsoft Visual Studio provides a convenient way to publish Azure Web Apps directly from the IDE. However, the deployment of an application is more than just publishing the bits for execution. An application deployment needs repeatable steps that you execute the same way each time you deploy the application. These steps start with building code and end with an application deployed to a production environment. 

In this post, I outline what you should consider when deploying an application as an Azure Web App.

## Use a build server

Build servers use the code you've checked into source control and compile it using an independent machine without any of the specialized tooling that developer machines typically have.  Using a build server ensures the application includes all dependencies when deployed to destination servers.  This avoids the "works on my machine" (WOMM) scenarios.

### Build once, deploy many

To ensure a consistent deployment experience, it's recommended you build the artifacts once and then use them to deploy to all environments. The build-per-branch approach introduces risk in that it doesn't guarantee that the same code gets tested in each environment. In theory, pull requests (PRs) *should* prevent unwanted or unanticipated changes, but the risk of bonus code getting introduced is still there.  

### Versioning scheme

Artifact version numbers should follow the [Semantic Versioning (SemVer)](https://semver.org/) structure. SemVer uses pre-release tags and is a recognized naming convention for versions that are not meant to be deployed to production environments. Only artifacts without pre-release tags should get released to a production environment.

## Choose an App Service Plan with slot support

Azure has a number of [App Service Plans](https://learn.microsoft.com/en-us/azure/app-service/overview-hosting-plans) to choose from.  For the best deployment experience, select an App Service Plan that supports deployment slots.  Deployment slots let you perform blue/green style deployments, so you can perform smoke tests against the newly deployed application without affecting the one currently in use.

## Use an automated deployment solution

Automated deployment solutions allow you to configure deployment processes that deploy the application in a repeatable, consistent fashion from environment to environment.  The choice of solution will depend on the level of complexity your deployment requires.  For simple deployments, something like GitHub Actions may be sufficient.  Larger, more complex deployments would need a solution like Octopus Deploy.

### Steps to include in your process

Through the years, I've worked with many customers.  In my experience, these are the steps you should include in your deployment process for an Azure Web App:

- Deployment start notification
- Deploy the Azure Web App to the staging slot
- Smoke test staging slot
- Approval step before swapping staging and production slots (in applicable environments, i.e., production)
- Swap staging and production slots
- Deployment successful notification
- Depoloyment failed notification (on failure)

#### Deployment start notification

Notification will depend on what communication mechanisms your organization uses.  The most popular are Slack and Microsoft Teams, however, email is often a universal option. If you're in a mature DevOps organization that deploys to lower-level environments many times a day, you may not need a notification for every environment. However, notifications are still highly recommended for higher environments, like staging and production.

#### Deploy the Azure Web App to the staging slot

Deployment slots are a powerful option. Deployment slots let you deploy the application without affecting the version currently in use. Using a slot gives you a version of the application that you can test to make sure it's working as desired before it's released (swapped) to the production slot.

#### Smoke test staging slot

You should create smoke tests to make sure the application is working as desired.  At a minimum, you should make a call to the application to ensure the entry point is responding with a 200 code.  This test will verify that the application spins up correctly and does not encounter any errors due to configuration.

#### Approval step before swapping staging and production

Similar to the start notification, an approval step may not be necessary for lower-level environments. For auditing purposes, higher-level environments should contain a step that captures who approved a deployment to proceed and when. The approval can be as simple as a manual intervention step in Octopus or an integration with IT Service Management (ITSM) software, like [Jira Service Management](https://www.atlassian.com/software/jira/service-management) or [ServiceNow](https://www.servicenow.com/).

#### Swap staging and production slots

Another advantage of using the deployment slots feature is the ability to swap between slots. Rather than redeploying the same version to the production slot, simply swap what's in production with what's in staging.  This gives you the flexibility to swap back if an issue surfaces with the new version.

#### Deployment successful notification

Deployment success or complete notifications are helpful so staff and stakeholders are aware when the new version is available.  This notification can also inform testing groups that the new version is available for testing.

#### Deployment failed notification

Failure notifications are incredibly important, especially in the production environment. In addition to the methods mentioned, failure notifications can often include notifications via incident management systems, like [PagerDuty](https://www.pagerduty.com/).

## Implement best practices using the Octopus AI Assistant

The Octopus AI Assistant is a Google Chrome extension that lets you use the power of Artificial Intelligence to create projects in Octopus Deploy. Supplying the Octopus AI Assistant with a prompt:

```
Create an Azure web app project called "My Web App 1"
```
Octopus will create a new Azure Web App project that includes the steps I've outlined in this post.  The project will also come with everything you need to get started right away, including project variables, an Azure Service Principal account, and runbooks to create the Azure resources.  The Octopus AI Assistant can work with more complex prompts, like:

```
Create an Azure web app project called "My Web App" Create a Lifecycle called "My Lifecycle" and set it as the default for the new project. Create a Phase in that lifecycle called "Phase 1" and add the Environment named "Development". Add a second phase called "Phase 2" and add the environments named "Test" and "Production".
```
In this case, Octopus will create a project similar to the first, but it will also create a new lifecycle with 3 phases and environments assigned accordingly.

## Conclusion

This post discussed what you should consider when deploying an Azure Web App.  In my experience, the steps listed make a deployment of an Azure Web App successful.  With the Octopus AI Assistant, you can create projects with best practices baked in at the start.

Happy deployments!