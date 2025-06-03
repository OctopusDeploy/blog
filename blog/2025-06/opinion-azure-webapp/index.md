---
title: Opinions on deploying an Azure Web App
description: This post discusses my opnion on how to deploy an Azure Web App.
author: shawn.sesna@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - tag
---

Pushing up code for an Azure Web App is not a particulary difficult task.  Microsoft Visual Studio provides a convenient way to publish Azure Web Apps directly from the IDE.  However, a deployment of an application is more than just publishing the bits for execution.  An application deployment needs to consist of repeatable steps that are executed the exact way each time the application is deployed.  These steps start with building code and end with an application deployed to a Production environment.  In this post, I will outline what should be considered when deploying an application as an Azure Web App.

## Use a Build server
Build servers use the code that has been checked into source control and compile it using an independent machine without any of the specialized tooling developer machines typically have.  Using a build server ensures the application includes all dependencies to run when deployed to destination servers.  This avoids the Works On My Machine (WOMM) scenarios.

### Build once, deploy many
To ensure a consistent deployment experience, it is recommended that artifacts be built once and used to deploy to all environments.  The build-per-branch approach introduces risk in that it doesn't guarnatee that the same code is tested in each environment.  In theory, Pull Requests (PR) _should_ prevent unwanted or unanticipated changes, but the risk of bonus code getting introduced is still there.  

### Versioning scheme
Artifact version numbers should follow the [Semantic Versioning (SemVer)](https://semver.org/) structure.  Use of pre-release tags are included in SemVer and are a recognized naming convention for versions that are not meant to be deployed to Production environments.  As the name suggests, artifacts with pre-release tags are not meant for Production, only artifacts without pre-release tags should be released to a Production environment.

## Choose an App Service Plan with Slot support
Azure has a number of [App Service Plans](https://learn.microsoft.com/en-us/azure/app-service/overview-hosting-plans) that you can choose from.  For the best deployment experience, choose an App Service Plan that supports the use of Deployment Slots.  Deployment slots give you the ability to perform blue/green style deployments allowing you to perform smoke tests against the newly deployed application without affecting the one that is currently in use.

## Use an automated deployment solution
Automated deployment solutions allow you to configure deployment processes that deploy the application in a repeatable, consistent fashion from environment to environment.  The choice of solution will depend on the level of complexity your deployment requires.  For simple deployments, something like GitHub Actions may be sufficient.  Larger, more complex deployments would need a solution like Octopus Deploy.

### Steps to include in your process
Through the years I have worked with quite a number of customers.  In my experience, these are the steps you should include in your deployment process for an Azure Web App:
- Deployment start notification
- Deploy the Azure Web App to the Staging slot
- Smoke test Staging Slot
- Approval step before swapping Staging and Production slots (in applicable environments - i.e. Production)
- Swap Staging and Production slots
- Deployment successful notification
- Depoloyment failed notification (on failure)

#### Deployment start notification
Notification will depend on what communication mechanisms your organization uses.  The most popular are Slack or Microsoft Teams, however, email is usually a universal option.  If you are in a mature DevOps organization that deploys to lower level environments many times a day, you may not need a notification for every environment, it is highly recommended for higher ones such as Staging and Production.

#### Deploy the Azure Web App to the Staging slot
Deployment Slots are a powerful option, they allow you to deploy the application without affecting the version that is currently in use.  Using a slot gives you a version of the application that you can test to make sure it is working as desired before it is released (swapped) to the Production slot.

#### Smoke test Staging Slot
You should create a smoke test(s) to make sure the application working as desired.  At a bare minimum, a call to the application should be made to make sure the entry point is responding with a 200 code.  This test will verify that the application spins up correctly and is not encountering any errors due to configuration.

#### Approval step before swapping Staging and Production
Similar to the start notification, an approval step may not be necessary for lower level environments.  For auditing purposes, higher level environments should contain a step that captures the person who approved the deployment to proceed.  The approval can be as simple as a Manual Intervention step in Octopus Deploy or an integration with an IT Service Management (ITSM) software such as [Jira Service Management](https://www.atlassian.com/software/jira/service-management) or [ServiceNow](https://www.servicenow.com/).

#### Swap Staging and Production slots
Another advantage of using the Deployment Slots feature is the ability to swap between slots.  Rather than redeploying the same version to the Production slot, simply swap what is in Production with what is in Staging.  This gives you the flexibility to swap it back in the event an issue surfaces with the new version.

#### Deployment successful notification
Deployment success or complete notifications are helpful so staff and or stakeholders are aware when the new version is available.  This notification can also serve as an indicator to testing groups that the new version is available to be tested for deployments to environments such as Staging.

#### Deployment failed notification
Failure notifications are incredibly important, especially in the Production environment.  In addition to the methods previously mentioned, failure notifications can often include notifications via incident management systems such as [PagerDuty](https://www.pagerduty.com/).

## Conclusion
This post discussed what you should consider when deploying an Azure Web App.  In my experience, the steps listed are required to make a deployment of an Azure Web App successful.

Happy deployments!
