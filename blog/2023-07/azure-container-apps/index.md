---
title: How to deploy Azure Container Apps
description: Learn how to deploy Azure Container Apps with Octopus Deploy.
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
 - Events and Webinars
 - Engineering
---

Containerizing your application is considered the modern day approach to application development.  The most popular platform for hosting containers is Kubernetes.  However, not everyone wants or needs to know how to operate a Kubernetes cluster, they just want to deploy their apps.  Azure Container Apps deploys containers to a Kubernetes cluster that is managed by Azure.  This allows developers to focus on what they do best, write code.  In this post, I'll demonstrate how to deploy the [eShopOnWeb](https://github.com/OctopusSamples/eShopOnWeb) application to Azure Container Apps with Octopus Deploy.


## What you'll need

Before you can deploy to Azure Container Apps, you'll need a few things configured:
- A Container Registry for your containers
- A build server to push your containers to the registry
- An Octopus Docker Container Registry External Feed
- An Azure Container Apps Environment

### Configure Container Registry

The first thing you'll need is a place to push your containers to.  Octopus is compatible with a variety of Docker Container Registries, this post will make use of an Azure Container Registry.  If you already have a Docker Container Registry configured, you may skip to the next section.

To create an Azure Container Registry, click on `Create a resource` in the Azure Portal

![Azure Create Resource](azure-create-resource.png)

Type `Azure Container Registry` in the search box and choose `Container Registry` from the list

![Azure Create Container Registry](azure-create-container-registry.png)

Fill in the required inputs and click `Create`

![Azure Create Container Registry Create screen](azure-create-container-registry-final.png)

Once the resource has been created, copy the Login server information, you'll need it for when you configure the Docker build task and the Octopus Deploy External Feed.

![Newly created ACR resource](new-azure-container-registry.png)


### Configure Build Server

Now that you have a container registry, you'll need to configure a build server to push your containers to the registry.  For this post, I'll be using Azure DevOps to demonstrate building the eShopOnWeb containers and pushing them to an Azure Container Registry.  eShopOnWeb consists of two containers:
- API
- Web

My build will consist of two Docker build tasks, both using the `buildAndPush` command.  

#### Add Docker build task
Filter the list of tasks by typing `docker` into the search bar, choose the Docker task

![Add Docker task to build](azure-devops-add-docker-task.png)

The Docker task will need a Docker Registry Service Connection.  Select `New` and fill in the form details.
- Docker Registry: The Login server from your ACR in URL format = `https://<Login server>`, example https://octopusdeploy.azurecr.io
- Docker ID: User ID for logging into your ACR.  I chose to use an App Registration so the value of the ID is the Application (client) ID of the App Registration
- Docker Password: Password for the User ID.  If you're using an App Registration, this is the Secret for the App Registration
- The rest of the fields can be default

![New Azure DevOps Service Connection](azure-devops-service-connection.png)

For the API Docker build task, use the following values:
- Container repository: eshop-api
- Command: buildAndPush
- Dockerfile: src/PublicApi/Dockerfile
- Build context: .
- Tags: `$(Build.BuildNumber)` `(Optional, I set my Build format number to $(Year:yyyy).$(Month).$(DayOfYear)$(Rev:r))`

![Docker build task](azure-devops-docker-task.png)

The Web Docker build task will be identical with the following differences:
- Container repository: eshop-web
- Dockerfile: src/Web/Dockerfile

Once you queue a new build, it should push your containers into your registry.

![Container images in ACR](azure-container-registry-containers.png)

### Configure Octopus Deploy External Feed
Now that the containers are hosted within ACR, you need to configure an External Feed within Octopus.  Navigate to **Library**->**External Feeds** and click **ADD FEED**

![Navigate to Libary, External Feed](octopus-library-external-feed.png)

Select the feed type, for this post, I'm using `Azure Container Registry`

![Select Azure Container Registry](octopus-azure-container-registry.png)

Similar to the Service Connection in Azure DevOops, add the feed details
- Name: A meaningful name for the External Feed
- URL: `https://<Login Server>` - example: https://octopusdeploy.azurecr.io
- Credentials:
  - Feed Username: User to connect to the registry.  I used an App Registration.
  - Feed Password:  Password for the user account.  Mine was the Secret for the App Registration.

Once that information is filled in, click on **SAVE AND TEST**.  Enter the image name, full or partial, you want to search for.

![Test External Feed](ocotopus-save-and-test.png)

### Configure Project Variables
This post assumes you are familiar with how to create an Octopus Project and will not cover that topic.  For the eShopOnWeb Octopus project, configure the following variables:
- Project.Azure.Account: An Azure Service Prinicipal account variable. (Optional, the templates used can use Managed Identity)
- Project.Azure.ContainerApp.Api.Name: Value to set for the Container Name, ex: #{Octopus.Environment.Name}-eshop-api
- Project.Azure.ContainerApp.Environment.Name: Name of the Azure Container App Environment to create/use, ex: #{Octopus.Environment.Name | ToLower}
**Note:** Azure Container App Environment names **must** be lower case.
- Project.Azure.ContainerApp.Web.Name: Value to set for the Container Name, ex: #{Octopus.Environment.Name | ToLower}-eshop-web
- Project.Azure.Region.Code: The short name for the Azure Region, ex: westus3
- Project.Azure.ResourceGroup.Name: Name of the Azure Resource Group to create the resources in.
- Project.Catalog.Database.Name: Name of the Catalog database for eShopOnWeb, ex: #{Octopus.Environment.Name}-eshop-catalog
- Project.Identity.Database.Name: Name of the Identity database for eShopOnWeb, ex: #{Octopus.Environment.Name}-eshop-identity
- Project.SQL.DNS: The DNS name for the SQL database server, ex: `<YourAzureSQLServer>.database.windows.net`
- Project.SQL.User.Name: Name of the SQL account for database access, ex: eShopUser
- Project.SQL.User.Password: Password for the SQL account

![Octopus project variables](octopus-project-variables.png)

### Configure Deployment Process
With the External Feed and project variables configured, you can now configure the deployment process.  For the eShopOnWeb application, the process consists of the following steps:
- Azure - Create Container App Environment
- Azure - Deploy api Container App
- Get API DNS
- Azure Deploy web Container App

#### Azure - Create Container App Environment
To deploy to Azure Container App, you must first have an Azure Container App Environment.  Add the [Azure - Create Container App Environment](https://library.octopus.com/step-templates/9b4b9fdc-2f97-4507-8df5-a0c1dd7464a5/actiontemplate-azure-create-container-app-environment) Community Step Template to your process.  The template will first check to see if it already exists, if not, create it.  This template takes the following input:
- Azure Resource Group Name: #{Project.Azure.ResourceGroup.Name}
- Azure Account Subscription Id: #{Project.Azure.Account.SubscriptionNumber}
- Azure Account Client Id: #{Project.Azure.Account.Client}
- Azure Account Tenant Id: #{Project.Azure.Account.TenantId}
- Azure Account Password:  #{Project.Azure.Account.Password}
- Container App Environment Name:  #{Project.Azure.ContainerApp.Environment.Name}
- Azure Location: #{Project.Azure.Region.Code}

![Azure - Create Container App Environment](octopus-azure-create-environment.png)

The template sets an output variable called `ManagedEnvironmentId` which is the Id value of the Environment it created or found if it already exists.

#### Azure - Deploy api Container App
Add an [Azure - Deploy Container App](https://library.octopus.com/step-templates/db701b9a-5dbe-477e-b820-07f9e354f634/actiontemplate-azure-deploy-container-app) Community Step Template to your process

- Azure Resource Group Name: #{Project.Azure.ResourceGroup.Name}
- Azure Account Subscription Id: #{Project.Azure.Account.SubscriptionNumber}
- Azure Account Client Id: #{Project.Azure.Account.Client}
- Azure Account Tenant Id: #{Project.Azure.Account.TenantId}
- Azure Account Password:  #{Project.Azure.Account.Password}
- Container App Environment Name:  #{Octopus.Action[Azure - Create Container App Environment].Output.ManagedEnvironmentId} **Note:**  This uses the output variable from the Azure - Create Container App Environment step
- Azure Location: #{Project.Azure.Region.Code}
- Container Name: #{Project.Azure.ContainerApp.Api.Name}
- Container Image: Choose eshop-api from feed list.
- Environment Variables:
```json
[
  {
    "name": "ConnectionStrings__CatalogConnection",
    "value": "Server=#{Project.SQL.DNS},1433;Integrated Security=false;Initial Catalog=#{Project.Catalog.Database.Name};User Id=#{Project.SQL.User.Name};Password=#{Project.SQL.User.Password};Trusted_Connection=false;Trust Server Certificate=True;"
  },
  {
    "name": "ConnectionStrings__IdentityConnection",
    "value": "Server=#{Project.SQL.DNS},1433;Integrated Security=false;Initial Catalog=#{Project.Identity.Database.Name};User Id=#{Project.SQL.User.Name};Password=#{Project.SQL.User.Password};Trusted_Connection=false;Trust Server Certificate=True;"
  },
  {
    "name": "ASPNETCORE_URLS",
    "value": "http://+:80"
  },
  {
    "name": "MyTest",
    "secretref": "mysecret"
  },
  {
    "name": "ASPNETCORE_ENVIRONMENT",
    "value": "Development"
  }
]
```
Secrets: 
```json
[
  {
    "name": "mysecret",
    "value": "#{Project.SQL.User.Password}"
  }
]
```
- Container Ingress Port: 80
- External Ingress: Unchecked (False)

![Azure - Deploy Container App](octopus-azure-deploy-container.png)

#### Get API DNS
The eShopOnWeb web container needs the URL to the API container.  This step runs the following code to retrieve that value and sets an output variable. This code assumes that the Az PowerShell modules aren't installed, so it does it dynamically.

```powershell
$PowerShellModuleName = "Az.App"
$LocalModules = (New-Item "$PWD/modules" -ItemType Directory -Force).FullName

Save-Module -Name $PowerShellModuleName -Path $LocalModules -Force
$env:PSModulePath = "$LocalModules$([IO.Path]::PathSeparator)$env:PSModulePath"

# Import modules
Import-Module Az.App

# Get reference to the container
$apiContainerApp = Get-AzContainerApp -Name "$($OctopusParameters["Project.Azure.ContainerApp.Api.Name"])" -ResourceGroupName $OctopusParameters["Project.Azure.ResourceGroup.Name"]

Set-OctopusVariable -name "DNS" -value $apiContainerApp.IngressFQDN
```

#### Azure Deploy web Container App
- Azure Resource Group Name: #{Project.Azure.ResourceGroup.Name}
- Azure Account Subscription Id: #{Project.Azure.Account.SubscriptionNumber}
- Azure Account Client Id: #{Project.Azure.Account.Client}
- Azure Account Tenant Id: #{Project.Azure.Account.TenantId}
- Azure Account Password:  #{Project.Azure.Account.Password}
- Container App Environment Name:  #{Project.Azure.ContainerApp.Environment.Name}
- Azure Location: #{Project.Azure.Region.Code}
- Container Name: #{Project.Azure.ContainerApp.Web.Name}
- Container Image: Choose eshop-web from feed list.
- Environment Variables:
```json
[
  {
    "name": "ConnectionStrings__CatalogConnection",
    "value": "Server=#{Project.SQL.DNS},1433;Integrated Security=false;Initial Catalog=#{Project.Catalog.Database.Name};User Id=#{Project.SQL.User.Name};Password=#{Project.SQL.User.Password};Trusted_Connection=false;Trust Server Certificate=True;"
  },
  {
    "name": "ConnectionStrings__IdentityConnection",
    "value": "Server=#{Project.SQL.DNS},1433;Integrated Security=false;Initial Catalog=#{Project.Identity.Database.Name};User Id=#{Project.SQL.User.Name};Password=#{Project.SQL.User.Password};Trusted_Connection=false;Trust Server Certificate=True;"
  },
  {
    "name": "ASPNETCORE_URLS",
    "value": "http://+:80"
  },
  {
    "name": "baseUrls__apiBase",
    "value": "https://#{Octopus.Action[Get API DNS].Output.DNS}"
  },
  {
    "name": "ASPNETCORE_ENVIRONMENT",
    "value": "Development"
  }
]

```
- Secrets: (blank)
- Container Ingress Port: 80
- External Ingress: Checked (True)

When done, your process should look like this

![Octopus Deployment Process](octopus-deployment-process.png)

### Deployment
Once the deployment is complete, it should have created the following Azure resources:
- Catalog database
- Identity database
- Azure Container App Environment
- eshop-api container app
- eshop-web container app

![Azure resources](azure-resources.png)

Click into the `eshop-web` Container App, then click on the Application Url to see the application running

![eShopOnWeb application running on Azure Container Apps](eshop-on-web.png)

## Conclusion

In this post, I demonstrated how to deploy the eShopOnWeb application to Azure Container Apps using Octopus Deploy.  Happy deployments!

## Learn more

- [link](https://www.example.com/resource)
- [repo with examples if used](https://www.github.com/repo)

## Register for the webinar: {webinar title here}

Short webinar description here, for example: A robust rollback strategy is key to any deployment strategy. In this webinar, we’ll cover best practices for IIS deployments, Tomcat, and full stack applications with a database. We’ll also discuss how to get the rollback strategy right for your situation. 

We're running 3 sessions of the webinar, from {webinar dates here, for example: 4 November to 5 November, 2021.}

<span><a class="btn btn-success" href="/events/rollback-strategies-with-octopus-deploy">Register now</a></span>

## Watch the webinar: {webinar title here}

<iframe width="560" height="315" src="https://www.youtube.com/embed/F_V7r80aDbo" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

We host webinars regularly. See the [webinars page](https://octopus.com/events) for details about upcoming events, and live stream recordings.

Happy deployments!
