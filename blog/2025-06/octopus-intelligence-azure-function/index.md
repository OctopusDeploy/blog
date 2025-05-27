---
title: Opinions on Azure Function deployments
description: Learn what Octopus Deploy considers a best practice deployment of an Azure Function.
author: shawn.sesna@octopus.com
visibility: private
published: 3020-01-01
metaImage: to-be-added-by-marketing
bannerImage: to-be-added-by-marketing
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: <!-- see https://github.com/OctopusDeploy/blog/blob/master/tags.txt for a comprehensive list of tags -->
 - DevOps
 - Company
 - Product
 - Engineering
---

During my tenure as a Solutions Engineer (SE) for Octopus Deploy, I have come across many different ways to perform a deployment of software.  Some of these solutions are quite inventive, while others have been less than optimal.  In this post, I will go over what I believe a deployment of an Azure Function should look like.

### The ideal deployment process for an Azure Function 
Deployment of an Azure Function can be accomplished using a range of different tools and or methods.  Azure Functions can be manually deployed by publishing through Visual Studio or similar code editor, using the Azure CLI, or using fully automated tool such as GitHub Actions or Ocotpus Deploy.  Using an automated tool will ensure that all steps are executed the exact same way in every environment, giving you a reliable and consistent deployment experience.

Regardless of what method you use, your deployment should consist of the following steps:
- Notification when a deployment is starting
- Deploy the Function to a Staging slot
- Smoke test(s)
- Approval step (Production environment)
- Swap the Staging and Production slots
- Notification the deployment was successful
- Notification the deployment failed (only on failure)

#### Notification when a deployment is starting
A deployment event should never be a surprise, especially to the Production environment.  An argument can be made that mature organizations that deploy several times a day to lower level environments don't necessarily need notifications for environments such as Development as too many notifications result in noise that nobody pays attention to.  The Production environment, however, should always have some sort of notification.  

Notifications can take place in many different ways such as email, Slack, Microsoft Teams, Amazon Chime, or even SMS messages.  Team members, stakeholders, and or management should be among the list of people to receive a notification that a deployment is beginning.

#### Deploy the Function to a Staging slot
The Azure Functions offering from Microsoft comes with with a freature called [Deployment Slots](https://learn.microsoft.com/en-us/azure/azure-functions/functions-deployment-slots).  It is common to create at least one additional Deployment Slot, usually named Staging, and deploy to it first to allow for testing before releasing the new version to that environment.

It is worth mentioning that the deployment slots capability will vary based on the Azure App Plan that you assign to your function (consult the [Hosting options](https://learn.microsoft.com/en-us/azure/azure-functions/functions-deployment-slots) table to determine if your Azure App Plan has deployment slots).

#### Smoke test(s)
After the function has been uploaded to Azure, you should perform some type of test to ensure that the function is operating properly, commonly referred to as a smoke test.  Smoke tests can be as simple as making sure the function returns a 200 code when called or something more complex like providing a JSON payload and testing to make sure the function processes it according to its specifications.  When used in conjuction with Deployment Slots, these tests can help identify issues before it has a chance to be a problem.

#### Approval step (Production environment)
For at least the Production environment, include a step that requires someone to take responsibility for the release into that environment.  An approval step can be something manual where the deployment is paused until someone clicks a button, or an integration with an IT Service Management (ITSM) tool such as [ServiceNow](https://www.servicenow.com) or [Jira Service Management](https://www.atlassian.com/software/jira/service-management).  The approval step can also with any auditing concerns your organizatio may have or are subject to.

#### Swap the Staging and Production slots
An additional capability of the Deploynment Slot feature is the ability to swap from one slot to another.  Swapping the slot makes your deployment akin to the blue/green deployment style while also providing a method to switch back the last known version of the function easily.  This feature is preferable over deploying to the Production Deployment Slot.

#### Notification the deployment was successful
Similar to the reasoning of notification when a deployment starts is the communcation that the deployment was successful.  This type of notification could be what is used to let testers know they can begin or stakeholders know when a bug has been fixed and is availalbe for use.

#### Notification the deployment failed (only on failure)
Notification that a deployment has failed is equally as important, if not more so.  Failure notifications should take advantage of whatever features are avaialable to make them stick out.  Important flags in email, red text notifications within instant messenger programs, whatever is necessary to make the failure noticed as soon as possible so it can be investigated and or mediated.  In such cases, deployment failures may also need to employ additional notifications such as a PagerDuty event, depending on severity.

## Conclusion
In this post I discussed the best practices for configuring an Azure Function deployment process.

Happy deployments!
