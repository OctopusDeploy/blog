---
title: Deploying to dynamically provisioned infrastructure
description: How to dynamically include new infrastructure during a deployment
author: shawn.sesna@octopus.com
visibility: public
bannerImage: octopus-dynamic-targets.png
bannerImageAlt: Deploying to dynamically provisioned infrastructure
metaImage: octopus-dynamic-targets.png
published: 2020-02-10
tags:
 - DevOps
 - Kubernetes
---

![Deploying to dynamically provisioned infrastructure](octopus-dynamic-targets.png)

Using project triggers, it’s possible to have your application deployed to dynamically created deployment targets.  This works especially well when you have an application that is configured with scaling capabilities.  As more servers are added to handle the load, your application is automatically deployed.  That being said, there are situations where the creation of your deployment target is part of your deployment process, and this can get a bit tricky as Octopus Deploy chooses the targets to deploy to when the deployment starts.  In this post, I show you how to include a dynamically added target as part of the deployment process.

## Example scenario

Let’s say we have an application that uses Kubernetes on Azure. We use an Azure Resource Manager (ARM) template to create our Kubernetes cluster, and then deploy our application to it.  Once the cluster is no longer needed, we tear it down to conserve cost.  This approach is useful when the type of cloud resource we need is expensive, and we only want it for a limited time. Our process looks something like this:

![](k8-deploy-process1.png)

We dynamically create a resource group, a Kubernetes cluster, then start deploying to it.

In order to add the Kubernetes cluster, we first need to add a post deployment script to the Create K8 cluster step.  Edit the step and click on the **CONFIGURE FEATURES** button:

![](octopus-project-k8s-step-configure-features.png)

Select Custom Deployment Scripts:

![](octopus-project-k8s-step-configure-features-scripts.png)

In the case of K8s, Octopus Deploy has created a helper cmdlet called [New-OctopusKubernetesTarget](https://octopus.com/docs/infrastructure/deployment-targets/dynamic-infrastructure/kubernetes-target).  For this article, I use a script that can add any type of target.

Expand the **Custom Deployment Scripts** section and enter the following into the **Post-Deployment Script** window.  This script will first check to see if it already exists, and if not, create the payload for the API call to add it:

```powershell
# Get current clustername
$kubernetesCluster = (Invoke-RestMethod -Method Get -Uri "$($OctopusParameters['Octopus.Web.BaseUrl'])/api/Spaces-1/machines/all" -Headers @{"X-Octopus-ApiKey"="$($OctopusParameters['Global.Octopus.ApiKey'])"}) | Where-Object {$_.Name -eq $OctopusParameters['Project.Azure.Kubernetes.ClusterName']}

# Check for null
if ($null -eq $kubernetesCluster)
{
  # Create payload for call
  $kubernetesClusterTarget = @{
  #Id= $null
  MachinePolicyId= "MachinePolicies-1"
  Name= $OctopusParameters['Project.Azure.Kubernetes.ClusterName']
  IsDisabled= $false
  HealthStatus= "Unknown"
  HasLatestCalamari= $true
  StatusSummary= $null
  IsInProcess= $true
  EndPoint= @{
  #Id= $null
  CommunicationStyle= "Kubernetes"
  Links= $null
  Authentication= @{
  AuthenticationType= "KubernetesAzure"
  AccountId= $OctopusParameters['Project.Azure.Account']
  ClusterName= $OctopusParameters['Project.Azure.Kubernetes.ClusterName']
  ClusterResourceGroup= $OctopusParameters['Project.Azure.ResourceGroup.Name']
  }
  AccountType= "AzureServicePrincipal"
  ClusterUrl= $null
  ClusterCertificate= $null
  SkipTlsVerification= $null
  DefaultWorkerPoolId= "WorkerPools-41"
  }
  Links= $null
  TenantedDeploymentParticipation= "Untenanted"
  Roles= @("OctoPetShop-K8")
  EnvironmentIds= @($OctopusParameters['Octopus.Environment.Id'])
  TenantIds= @()
  TenantTags= @()
  }

  # Convert to json
  $jsonBody = $kubernetesClusterTarget | ConvertTo-Json -Depth 10

  # Add cluster to deployment targets
  Invoke-RestMethod -Method Post -Uri "$($OctopusParameters['Octopus.Web.BaseUrl'])/api/Spaces-1/machines" -Body $jsonBody -Headers @{"X-Octopus-ApiKey"="$($OctopusParameters['Global.Octopus.ApiKey'])"}
}
```

![](octopus-project-k8s-step-powershell.png)

We then update the project settings to allow deployments to be created, even when targets do not exist:

![](project-target-settings.png)

Now we schedule our deployment and watch it go!

## Why are steps being skipped?

The result of your deployment will look like all of your steps for deploying to the Kubernetes cluster were skipped, despite successfully creating the Kubernetes cluster and registering it with Octopus Deploy. Deployment targets are chosen at the beginning of a deployment, and since the cluster hadn’t been registered to Octopus Deploy yet, Octopus determined there were no available targets to deploy to, and so all deployment steps were skipped.

![](steps-skipped.png)

## Including new targets

Adding a Health Check step to your process and configuring it to include new targets gets around this issue.

When configuring the Health Check, choose `connection-only test` for the Health Check Type and `Include new deployment targets` in the New Deployment Targets section.

![](configure-health-check.png)

After you have done this, the newly provisioned Kubernetes cluster will be included in the deployment, and the rest of our steps will deploy successfully!

![](successful-deployment.png)

## Summary

While not a common scenario, this post demonstrated how you can include targets that were created during the deployment process in your deployment.
