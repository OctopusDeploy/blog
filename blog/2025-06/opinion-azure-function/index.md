---
title: Opinions on Azure Function deployments
description: Learn what Octopus Deploy considers a best practice deployment of an Azure Function.
author: shawn.sesna@octopus.com
visibility: public
published: 2026-01-01-1400
metaImage: blog-opinions-on-azure-functions-deployments-2025-750x400.jpg
bannerImage: blog-opinions-on-azure-functions-deployments-2025-750x400.jpg
bannerImageAlt: Illustration of people having a discussion with speech bubbles, with the Azure Functions logo, a lightning bolt within code brackets, in the center. A green checkmark suggests validated or approved practices.
isFeatured: false
tags:
 - DevOps 
 - AI
 - Azure
 - Deployment Patterns
 - Serverless
---

During my tenure as a Solutions Engineer (SE) for Octopus Deploy, I've come across many different ways to deploy software. Some of these solutions are quite inventive, while others have been less than optimal. 

In this post, I cover what I believe a deployment of an Azure Function should look like.

## The ideal deployment process for an Azure Function 

You can deploy an Azure Function using a range of different tools and or methods. You can:

- Manually deploy Azure Functions by publishing through Visual Studio or a similar code editor
- Use the Azure CLI
- Use a fully automated tool, like GitHub Actions or Octopus Deploy  

Automated tools ensure you execute all steps the same way in every environment, giving you a reliable and consistent deployment experience.

Regardless of what method you use, your deployment should consist of the following steps:

- Notification when a deployment is starting
- Deploy the Function to a staging slot
- Smoke tests
- Approval step (production environment)
- Swap the staging and production slots
- Notification that the deployment was successful
- Notification that the deployment failed (only on failure)

### Notification when a deployment is starting

A deployment event should never be a surprise, especially to the production environment.  You can argue that mature organizations deploying several times a day to lower-level environments don't necessarily need notifications for all environments.  Too many notifications may lead to noise that nobody pays attention to, and important notifications getting lost.  The production environment, however, should always have some sort of notification.  

You can notify people using email, Slack, Microsoft Teams, Amazon Chime, or even SMS messages. Team members, stakeholders, and management should be among the people notified that a deployment is beginning.

### Deploy the Function to a staging slot

The Azure Functions offering from Microsoft includes a feature called [deployment slots](https://learn.microsoft.com/en-us/azure/azure-functions/functions-deployment-slots). It's common to create at least one additional deployment slot, usually named staging. You deploy to it first to allow for testing before releasing the new version to that environment.

It's worth noting that the deployment slots capability varies based on the Azure App Plan that you assign to your function. Consult the [hosting options](https://learn.microsoft.com/en-us/azure/azure-functions/functions-deployment-slots) table to determine if your Azure App Plan has deployment slots.

### Smoke tests

After the function has been uploaded to Azure, you should perform some type of test to ensure that the function is operating properly, commonly referred to as a smoke test. Smoke tests can be as simple as making sure the function returns a 200 code when called, or something more complex, like providing a JSON payload and testing to make sure the function processes it according to its specifications. When used in conjunction with deployment slots, these tests can help identify issues before they cause problems.

### Approval step (production environment)

For at least the production environment, include a step that requires someone to take responsibility for the release to that environment. An approval step can be something manual where the deployment gets paused until someone clicks a button, or an integration with an IT Service Management (ITSM) tool, like [ServiceNow](https://www.servicenow.com) or [Jira Service Management](https://www.atlassian.com/software/jira/service-management).  Highly secure organizations require some sort of approval to satisfy auditing requirements.

### Swap the staging and production slots

An additional capability of the deployment slot feature is the ability to swap from one slot to another.  Swapping the slot makes your deployment akin to the blue/green deployment style.  Swapping the slot is preferable to deploying to the production slot as it gives you a quick and easy way to revert to a known, working version.

### Notification that the deployment was successful

A notification that a deployment has completed successfully is also important.  This type of notification can let testers know they can start their work, or let stakeholders know when a bug has been fixed.

### Notification that the deployment failed (only on failure)

Notification that a deployment has failed is equally as important, if not more so.  Failure notifications should take advantage of whatever features are available to make them stand out. Important flags in email, red text notifications in instant messenger programs, or whatever makes the failure get noticed as soon as possible, so someone can investigate and mitigate the issue. Deployment failures may also need additional notifications, like a PagerDuty event, depending on severity.

## Use the Octopus AI Assistant to create projects according to best practices

Octopus has embraced AI and developed the Octopus AI Assistant. The Octopus AI Assistant gives you an easy-to-use, prompt-based system to create projects with best practices baked in.  For example, using the prompt:

```
Create an Azure Function project called "My Function 1" in space "My Space"
```

Octopus will create a new project including all the steps outlined in this post.  It will also create everything necessary to run the project, including runbooks to create the infrastructure, project variables, and a Service Principal Azure account.  If you'd rather use OIDC, all you need is a small tweak to the prompt:

```
Create an Azure Function project called "My Function 1" in space "My Space".  Make the Azure Account use OIDC.
```

## Conclusion

Part of my job as a Solutions Engineer is to help customers get the most out of Octopus. This often includes advising on deployment best practices, similar to what I outlined in this post. Customers can now use the Octopus AI Assistant to create projects quickly, easily, and with all the best practices baked in.

Happy deployments!