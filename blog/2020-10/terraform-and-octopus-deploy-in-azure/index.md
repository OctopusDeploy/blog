---
title: Infrastructure as Code in Azure with Terraform and Octopus Deploy
description:  In this blog post, we take a look at combining the power of Infrastructure-as-code using Terraform and Octopus Deploy to create resources and services in Azure.
author: michael.levan@octopus.com
visibility: private
published: 2199-10-10 
metaImage: 
bannerImage: 
tags:
 - Automation
 - DevOps
 - Azure
---

Infrastructure developers write code to automate the process of configuring cloud and on-premises infrastructure, in this post, I should your how to use Terraform and Octopus Deploy to deploy services to Azure.

## Prerequisites

To follow along with this blog post, you need the following:

- An Octopus Deploy server, either [on-premises](https://octopus.com/start/server) or a [cloud instance](https://octopus.com/start/cloud)
- An Azure subscription. If you don't already have one, you can sign up for a [30-day free trial](https://azure.microsoft.com/en-us/free/).
- An Azure Service Principal (app registration) that has access to create resources in your Azure subscription.
- Knowledge of [Terraform](https://www.terraform.io/intro/index.html) at a beginner-to-intermediate level.

## Why Use Octopus and Terraform Together

Terraform is an open-source [Infrastructure-as-Code](https://docs.microsoft.com/en-us/azure/devops/learn/what-is-infrastructure-as-code) platform created by [Hashicorp](https://www.hashicorp.com/) that is supported by default in Octopus Deploy. 

You can deploy Terraform resources to:

- Azure
- AWS
- On-premises (hardware and virtualized environments)

One of the major benefits of using Terraform in a continuous delivery and deployment tool is that you can focus on writing the code, not manually deploying it. Combining Octopus and Terraform allows you to automate the entire lifecycle.

## The Terraform Code

To create a resource or service in Azure, you need to write the HCL code. In this section, I show you the HCL code to create a Resource Group in Azure using Terraform.

### The Azure Terraform Provider

Whenever you interact with a Terraform provider, you need to specify some inputs and authentication in the code block. The provider that is used to interact with Azure is the [`azurerm` provider](https://www.terraform.io/docs/providers/azurerm/index.html).

There are four ways to authenticate to the `azurerm` Terraform provider:

- [Authenticating to Azure using the Azure CLI](https://www.terraform.io/docs/providers/azurerm/guides/azure_cli.html)
- [Authenticating to Azure using Managed Service Identity](https://www.terraform.io/docs/providers/azurerm/guides/managed_service_identity.html)
- [Authenticating to Azure using a Service Principal and a Client Certificate](https://www.terraform.io/docs/providers/azurerm/guides/service_principal_client_certificate.html)
- [Authenticating to Azure using a Service Principal and a Client Secret](https://www.terraform.io/docs/providers/azurerm/guides/service_principal_client_secret.html)

For the purposes of this blog post, we're using an Azure Service Principal.

The provider needs the following information:

- Azure Subscription ID
- Client ID
- Client Secret
- Tenant ID

There is also a `features` parameter, but it can be left blank.

The provider config block looks like the below code:

```
provider "azurerm" {
  subscription_id = "#{subscriptionID}"
  client_id       = "#{clientID}"
  client_secret   = "#{clientSecret}"
  tenant_id       = "#{tenantID}"
  
  features		  = {}
}
```

Notice of the subscription ID, client ID, client secret, and tenant ID have variables associated for the values. You'll go over setting up the variables in an upcoming section. 

## Creating the Azure Resource

The `resource` create operation will call upon the `azurerm_resource_group` resource type. The resource type contains two parameters needed in the config block:

- name - The name of the resource group you're creating
- location - The location where the resource group will reside (`eastus`, for example)

```
resource "azurerm_resource_group" "resourceGroup" {
  name     = "#{resourceGroupName}"
  location = "#{location}"
}
```

Once you have the provider and resource code, it should look like the below code snippet.

```
provider "azurerm" {
  subscription_id = "#{subscriptionID}"
  client_id       = "#{clientID}"
  client_secret   = "#{clientSecret}"
  tenant_id       = "#{tenantID}"
  
  features		  = {}
}

resource "azurerm_resource_group" "myterraformgroup" {
  name     = "#{resourceGroupName}"
  location = "#{location}"
}
```

## Authentication From Octopus Deploy to Azure

Now that the code is written that will be used to create a new Resource Group in Azure, you need a way to authenticate from Octopus Deploy to Azure. Luckily, Octopus Deploy has a way to create accounts for authentication to cloud and on-prem environments.

To create an Azure account..

Log into the Octopus Deploy portal and go to **Infrastructure** —> **Accounts**

![](images/1.png)

Click the green **ADD ACCOUNT** button and choose the **Azure Subscription** option.

![](images/2.png)

Add in all associated information for the Azure Service Principal you have that has permission to create resources in the Azure portal. To confirm that the Azure Service Principal works, click the **SAVE AND TEST** button.

![](images/3.png)

## Setting up a New Project in Octopus Deploy

Once the authentication is complete from Octopus Deploy to Azure, you can start thinking about how and where you want the Terraform runbook to exist. To ensure that the runbook is in it's own project, you can create the project with the Octopus Deploy UI.

### Creating a Project in Octopus Deploy

1. Log into the Azure portal and go to **Projects**.
2. Choose which project group you'd like to store the Project in and click the green **ADD PROJECT** button.
3. Create a new project and name it **TerraformAzure**.

Once the project is created, it's time to create the runbook.

### Creating Octopus Deploy Variables

Under the variables section of the project, you'll want to add in Project Variables. Because these values can differ based on the environment you're in, below is a code sample. The `Name` of the variables should match the value below because you will use them in the code later, but the values will be different for your environment.

```
AzureAuth         = AzureAuth Account
clientID          = guid_client_id
clientSecret      = client_secret
location          = eastus
resourceGroupName = your_resource_group_name
subscriptionID    = your_subscription_id
tenantID          = guid_tenant_id
```

![](images/4.png)

## Configure the Runbook

Since you're deploying a service in Azure and not code for an application, the most efficient action is to use a Runbook. The Runbook will give you the ability to use the Terraform step template and create the Resource Group.

## Creating a Runbook

1. Under the Project, go to **Operations** —> **Runbooks**.
2. Click **ADD RUNBOOK**.
3. Create a runbook and name it **ResourceGroup**.

## Add steps to the runbook

1. Navigate to the runbook, select **Process**, and click **ADD STEP.**
2. Click on the Terraform category.

3. Choose the **Apply a Terraform template** step.

![](images/6.png)

### Configure the Terraform Step

Depending on the environment you're running in, these steps could be different. For example, you could use a different Worker Pool than the default. These are the key steps to include for Terraform:

1. Under **Managed Accounts**, choose **Azure Account** and add the Azure account you created in the **Authentication to Octopus Deploy from Azure** section.
2. Under **Template**, choose **Template Source** and use the **Source code** option. Then, paste in the following code:

```
provider "azurerm" {
  subscription_id = "#{subscriptionID}"
  client_id       = "#{clientID}"
  client_secret   = "#{clientSecret}"
  tenant_id       = "#{tenantID}"
  
  features		  = {}
}

resource "azurerm_resource_group" "myterraformgroup" {
  name     = "#{resourceGroupName}"
  location = "#{location}"
}
```

As you can see, it uses the variables you created in the **Setting up Variables** section.

<<<<<<< HEAD
## Executing the Runbook
=======
## Run the Terraform code
>>>>>>> 8f7ff736c91f00a407ecb4a3d37e0307df257892

The configuration of the project, authentication, steps, and code is all complete. Now, it's time to see the code in action! 

1. Under **Runbooks**, you'll see the **ResourceGroup** runbook. Click **RUN**.
2. Select the environment that you'd like to run the runbook under and click **RUN**.

![](images/7.png)

You have successfully created a Resource Group in Azure using Octopus Deploy and Terraform.

## Conclusion

Combining the power of continuous deployment and infrastructure-as-code is key to any automated environment. Not only does it give you automation, but a place for fellow team members to collaborate, see what's happening, and understand the process instead of manually doing it yourself.
