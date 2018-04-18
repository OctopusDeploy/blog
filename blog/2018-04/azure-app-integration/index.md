---
title: Azure App Integrations - Integrating your apps with Azure Active Directory (AAD)
description: Integrated your apps and services with Azure can require working with 
author: rob.pearson@octopus.com
visibility: private
published: 2018-04-20
tags:
 - Azure
 - Deep Dive
---

Octopus Deploy integrates with popular cloud services like [Amazon Web Services (AWS)](https://aws.amazon.com/) and [Microsoft's Azure platform](https://azure.microsoft.com/) to make it easy to deploy your apps safely and securely. Integrating with Amazon is as simple as adding your AWS Access Key and Secret Key however, Azure requires a few more details and it's not immediately obvious how to set things up. This is enabled by Azure Active Directory (AAD) registered applications (or app registrations) but it can be tricky to setup so we're going to take a deeper look.

![Octopus Accounts](octopus-accounts.png "width=750")

Octopus requires four values which are utilised to authenticate with Azure and interact with it securely. 

* Azure Subscription ID
* AAD Tenant ID
* AAD Registered Application ID
* AAD Registered Application Password/Key

The first three values are GUIDs and the final one is a password. These values can be found via the Azure Portal or via Powershell. Personally, I stuggle to remember where to find all the values are found so we'll walk through the process now of finding/creating them.

<iframe width="560" height="315" src="https://www.youtube.com/embed/TODO" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

## App Registration in the Azure Portal

### Azure Subscrition ID

This one is easy. Navigate to the Azure Portal `Subscriptions` blade and pick the appropriate Subscription ID.

![Azure subscriptions](azure-subscriptions.png "width=500")

### AAD Tenant ID

This is another easy one. Navigate to the `Azure Active Directory` service and select the Properties blade. The Directory is your AAD Tenant ID. NOTE: This value is a GUID.

![Azure Active Directory properties](azure-ad-properties.png "width=500")

### AAD Registered Application ID and AAD Registered Application Password/Key

If you have created an AAD registered application, then it's relatively straight forward to note the Application ID and Password/Key. Navigate to to the Azure Active Directory service and select the App registrations blade. Make sure to click `View all applications` if you don't see anything there. If you have already created an registered application for integration, select the app and note the Application ID. 

![AAD registered applications](azure-ad-registered-apps.png "width=500")

If you haven't created an application, Click the New application registration and fill in the appropriate deails and then click `Save`. THen note the Application ID. 

![Create AAD registered app](azure-ad-create-registered-app01.png "width=500")

![Create AAD registered app](azure-ad-create-registered-app02.png "width=500")

Application Passwords are one-time genereated tokens and so if you don't already know your password, you'll need to generate a new one. If you haven't set one, the process is the same. 

Click the settings button and then the `Keys` blade. Add a new Password with a good description and click save. The password will be displayed after you click save. Be sure to note it as it won't be displayed again. 

![Set AAD registered app password](azure-ad-registered-app-password.png "width=500")

That's it, you now have your registered application ID and password.

### Permissions

The final step is to ensure your registered app has permission to work with your Azure resources. Navigate to the 'Resource Groups' service and select the resource group(s) that you want our application to access. 

![Resource Group permission](resource-group-perms01.png "width=500")

Next, select the Access Control (IAM) blade and if your app isn't listed, click the `Add` button. Select the appropriate role (Contributor is a good option) and search for your new application name. Select it from the search results and then click Save.  

![Resource Group permission](resource-group-perms02.png "width=500")

### Octopus Account

Now you can go ahead and create an Octopus Azure account and use it to deploy your applications safely and securely.

![Octopus Azure account](octopus-account.png "width=500")

## App Registration with Azure PowerShell

For those developers who prefer to run scripts, the following PowerShell script can retrieve the same values. It also creates 

```powershell
# Obviously, replace the following with your own values
$subscriptionId = "cd21dc34-73dc-4c7d-bd86-041284e0bc45"
$tenantId = "2a681dca-3230-4e01-abcb-b1fd225c0982"
$password = "correct horse battery staple"

# Login to your Azure Subscription
Login-AzureRMAccount
Set-AzureRMContext -SubscriptionId $subscriptionId -TenantId $tenantId

# Create an Octopus Deploy Application in Active Directory
Write-Output "Creating AAD application..."
$securePassword = ConvertTo-SecureString $password -AsPlainText -Force
$azureAdApplication = New-AzureRmADApplication -DisplayName "Octopus Deploy" -HomePage "http://octopus.com" -IdentifierUris "http://octopus.com" -Password $securePassword
$azureAdApplication | Format-Table

# Create the Service Principal
Write-Output "Creating AAD service principal..."
$servicePrincipal = New-AzureRmADServicePrincipal -ApplicationId $azureAdApplication.ApplicationId
$servicePrincipal | Format-Table

# Sleep, to Ensure the Service Principal is Actually Created
Write-Output "Sleeping for 10s to give the service principal a chance to finish creating..."
Start-Sleep -s 10

# Assign the Service Principal the Contributor Role to the Subscription.
# Roles can be Granted at the Resource Group Level if Desired.
Write-Output "Assigning the Contributor role to the service principal..."
New-AzureRmRoleAssignment -RoleDefinitionName Contributor -ServicePrincipalName $azureAdApplication.ApplicationId

# The Application ID (aka Client ID) will be Required When Creating the Account in Octopus Deploy
Write-Output "Client ID: $($azureAdApplication.ApplicationId)"

```

## Wrap-up

Now your'e good to go. You can enter all these values into Octopus or utilise them in your own application to integrate with Azure.  

If you've ever struggled to add an Octopus Azure account, we hope this makes it a lot easier for you. If there's another part of the Octopus that is puzzling you, let us know in the comments.

Don't forget to subscribe to our [YouTube](https://youtube.com/octopusdeploy) channel as we're adding new videos regularly. Happy deployments! :)
