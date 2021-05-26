---
title: Migrate an ASP.NET web app from IIS on-prem to an Azure App Service
description: How to migrate an existing ASP.NET web app from IIS on-premesis to an Azure App Service in the Cloud.
author: rob.pearson@octopus.com
visibility: public
published: 2022-06-02
metaImage: socialimage-github-actions-integration_2021.png
bannerImage: socialimage-github-actions-integration_2021.png
tags:
 - DevOps
 - Migration
---

![Migrate an ASP.NET web app from IIS on-prem to an Azure App Service](socialimage-github-actions-integration_2021.png)

Team are migrating their applications and infrastructure to the cloud. The natural migration path for ASP.NET web apps or services running on Microsoft's IIS web server on-prem is to App Services on Microsoft's Azure cloud platform. 

It's relatively straightforward to change the way your application is hosted but it's also important to ensure your CI/CD process is still repeatable and reliable. In this blog post, I'll walk through how to update the deployment process of an existing web application with Octopus Deploy shifting it from on-prem to the cloud. 

## Example application

We're using the Random Quotes web application as the example in this migration guide. This is a web application which retrieves and displays famous quotes randomly. It has an ASP.NET front end with a SQL Server backend. This is very simple however it helps illustrate the changes required. 

## Updating our deployment process

Screenshot1 TODO

Our existing on-prem based deployment process has two steps. It has a script step that executes a command line tool to update our database and a second step to deploy our web application package to IIS and configure the bindings. 

Screnshot2 TODO

Our cloud-based deployment process is similar however it's updated to suit the cloud. Our first step is still running a script step to udpaet our database but this time it's an Azure script step which means it's pre-authenticated and has access to the Azure CLI. The second step is an Deploy Azure App Service step. It captures everything required to deploy our web appliation package to an Azure app service including configuration file udpdates.

## Migrating to Octopus Cloud (optional)

One commmon pattern we've see with our customers as they shift away from virtual machines on-premises is that they also look to shift their CI/CD platform from on-prem tooling to the cloud. For example: companies shift from using TeamCity and Octopus on-prem to TeamCity Cloud and Octopus Cloud. 

I'll briefly outline how to accomplish this with Octopus Deploy. We recently launched Project export and import support to make it easy to move projects from one Octopus instance to another. This is self-service meaning that individual developers can do this themselves without the need for operations teams to help. 

Pre-requisite: Both of your Octopus instance need to be running Octopus 2021.1 (Build 7198) or newer. 

On-premises Ocotpus instance:

1. Navigate to the Projects dashboard. Select the more button (three virticle elipses) and then the Export Projects button. 
2. Select the project(s) that you'd like to export. This can be one or more. 
3. Set a password to protect the exported files.

Octopus Cloud instance:

1. Navigate to the Projects dashboard. Select the more button (three virticle elipses) and then the Import Projects button. 
2. Select your exported ZIP file from your on-prem server and enter your password.
3. Review the Import Preview and click the import button.

NOTE: There are limitations to the project import/export so I recommend reviewing our [documentation](TODO).

## Updated your infrastructure

In order to move to an Azure based deployment, we need to configure an Azure Account and add one or more Azure Web App deployment targets. The Azure account captures everything required to authenticate with an Azure subscription. This is then used to configure your Azure web app targets. 

Configuring an Azure account requires four specific IDs which is relatively complicated to configure. I won't go into the detail here and I highly recommend reviewing our docs to learn where to get the values to connect this integration. 

Next, we need to add one or more Azure Web App deployment targets. These are the App services that we will deploy our web application to. They are the replacements for our virtual machines running IIS. Configuring an Azure web app deployment target is straightfoward.

1. Navigate to Infrastructure -> Deployment Targets
2. Click the add target button and select Azure and then Azure web app. 
3. Configure the name, role and select the Azure account.

Repeat for your different subscriptions appropriate for your DEV and test infrastructure.

## Update your deployment process

We need to update our deployment process to support our move to the cloud. I could use [Ocotpus channels](TODO) to support the old and new deployment processes but I've imported this process so I don't think it's necessary. In this case, I'll delete the old IIS based deployment step. 

My database udpate step uses a script that uses the database connection string to connect to a database. So long as my worker has access to my target database, I don't need to change anything there.

The core part of the work to add and configure a new Deploy to Azure App Service deployment step. That said, this is relatively straightforward as we've already setup our infrastructure. 

One new thing we can do is wire-up our configuration updates directly in the step. In this case, we only have two configuration variables that require this. Random Quotes retrieve two configuration values that specific the application version and the environment it was deployed to. We can enter the following details in the app settings text box thing.

```json

```


## Update your configuration variables

We also need to update our Project variables to support the cloud. The primary thing we need to change here is our database connection string. In this case, I've composed my database connection string with individual variables which makes it easy to update. 

I can remove unused variables for ports and bindings etc. I could add host name or DNS details if needed but this doesn't apply in this case. 

## Explore runbooks

TODO: Write something about how runbooks can help a new Auzre app service. Examples are manually toggling blue/green deployments, provision and tear down DEV/TEST infrastructure on a schedule. i.e. Cost savings.

## Conclusion

Reword intro.