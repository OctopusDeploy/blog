---
title: Use the new Deploy an Azure App Service Step
description: Learn how to use the new Azure App Service Step in Octopus Deploy
author: terence.wong@octopus.com
visibility: public
published: 2021-10-04-1400
bannerImage: 
metaImage:
bannerImageAlt: 
isFeatured: false
tags:
- Azure
---


Octopus Deploy provides process steps to deploy to Microsoft Azure. A new step, deploy an Azure App Service, is now available to deploy containers to an Azure App service.

To do this, you will need:

- An Octopus Deploy instance with a project and environment
- An Azure Account
- to link the [Azure Account to the Octopus Deploy instance](https://octopus.com/docs/infrastructure/accounts/azure#azure-service-principal).

Set up the Azure Web App that you will deploy by going to your resource group. Click **{{Create, Web App}}**. Give the web app a name and check 'Docker Container' for the publish setting. Select the appropriate location and create the web app. You will see an option to go to the resource. The URL will be the address of the hosted Web App.

![Azure Web App Home](azure-web-app-home.png)

In the Octopus Deploy instance, go to **{{infrastructure, Deployment Targets, Add Deployment Target, Azure, Azure Web App}}** 

![Add deployment target](add-deployment-target.png)

Populate the following fields:

- Environments - the environment you wish to deploy to
- Target Roles - the role that will identify the deployment target, you may need to create one if it doesn't exist
- Account - the linked Azure Account in Octopus Deploy
- Azure Web App - your Web App that you created earlier.

Go to **{{Library, External Feeds, Add Feed}}**. Populate the following fields:

- Feed Type - Docker Container Registry
- Name - Give the feed a name

This step activates the public docker registry feed that we will use later. Click Save

Add the deploy an Azure App service step in your project process.

![Octopus Azure deploy step](deploy-an-azure-app-service-step.png)

Populate the fields with the following values:

- Worker Pool - Runs on a worker from a specific worker pool: Hosted Ubuntu
- On Behalf Of - the role you created in the deployment target step (mine is azure)
- Container Image - Runs inside a container on a worker: Container Registry: docker Image: octopusdeploy/worker-tools:3.2.0-ubuntu.18.04
- Package -Deploy from a container image Package feed: docker Package ID: octopussamples/randomquotes

In this example, we deploy a sample docker image hosted on Docker Hub. The image below shows the result of my configuration. Click Save.


![Octopus Azure deploy step configuration](deploy-process-step-config.png)

Click Create Release and click through to the deploy button to deploy the Web App to Azure.

![Deploy Success](deploy-success.png)

Navigate to the URL of your Web App - [your-url].azurewebsites.net

![Random Quotes](randomquotes.png)


In this blog, you have set up an Octopus Deploy Project to deploy a Web App using the new Deploy Azure App Service step.

Happy Deployments!


