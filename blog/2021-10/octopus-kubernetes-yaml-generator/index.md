---
title: Octopus Deploy's Kubernetes YAML generator
description: Learn how to use our Kubernetes YAML generator, and deploy a Kubernetes cluster.
author: terence.wong@octopus.com
visibility: public
published: 2021-10-26-1400
metaImage: blogimage-octopusdeployk8syamlgenerator-2021.png
bannerImage: blogimage-octopusdeployk8syamlgenerator-2021.png
bannerImageAlt: Octopus website window with a cog icon connecting it to another window generating the K8 YAML file
isFeatured: false
tags:
 - DevOps
 - Kubernetes
---

Kubernetes (K8s) is a powerful container orchestration tool. 

Kubernetes reads YAML files that define the resources you're deploying to. Of course, not everybody loves writing YAML. To make this easier, we released a tool that helps developers build YAML files for Kubernetes cluster deployments. 

You can find the tool at [https://k8syaml.com/](https://k8syaml.com/).

## Overview of our K8s YAML generator

A YAML file is a human-readable configuration file that tells Kubernetes how to provision and deploy a service. 

The left-hand side of our tool contains the various options for the YAML file. Each option has a drop-down menu that is populated. 

The right-hand side contains the YAML file that Kubernetes will use. 

Two features built into the K8s YAML tool are live updates and two-way sync.

### Live updates

When changing fields on the left-hand side of the tool, the YAML file on the right-hand side updates to match. Changing the resource type from **Deployment** to **StatefulSet** on the left-hand side will change the YAML file to match the new option. Live updates can add new options to the YAML file that weren't there before.

![Live Update Deployment](live-update-deployment.png "Live Update Deployment")*Deployment resource type*

![Live Update Stateful](live-update-stateful.png "Live Update Stateful")*Stateful set resource type*

### Two-way sync

You can edit the YAML in the tool by selecting the **EDIT YAML** button.

In the edit pane, you can edit any field. 

Below, I edited the name of the deployment to `test-deployment` and clicked **DONE**.

![Two Way Sync Edit](two-way-sync-edit.png "Two Way Sync Edit")

Two-way sync updates the left-hand side of the tool to match the edits. 

![Two Way Sync Test](two-way-sync-test.png "Two Way Sync Test")

### More information

There are several configurable options in a Kubernetes YAML file. Rather than explaining them in detail, the tool links to the official Kubernetes documentation. Most options have a **More information** link that takes you directly to the documentation.

![Kubernetes More Information](kubernetes-more-info.png "Kubernetes More Information")

## Use case

To show the YAML file in action, I use it to deploy a web application to an Azure Kubernetes Service with Octopus Deploy.  Feel free to follow along.

Fill out the fields according to the image, by populating the left-hand side. 

**Deployment** - Change the value of the app to `randomquotes`

**Containers** - Delete the nginx default container and add a new container with: 

- **Name**: `randomquotes`
- **Package Image**: `terenceocto/randomquotes-js`
- **Add Port**: `TCP:80`

Click **OK** to confirm.

The text on the right is the YAML file we use to deploy to Azure. Copy this file to use later.

![YAML generator](yaml-generator.png "YAML generator")


## Configuring an Azure account

You need to configure an Azure account and web application to act as a target for the deployment from Octopus. Other targets are possible, such as AWS, or Windows or Linux servers.

Next, you need to create an account in Azure, by navigating to the [Azure portal](https://portal.azure.com/). 

### Creating an Azure service principal with the Azure portal {#create-service-principal-account-in-azure}

<iframe width="560" height="315" src="https://www.youtube.com/embed/QDwDi17Dkfs" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

1. In the Azure Portal, open the menu, and navigate to **Azure Active Directory**, then **Properties**. 
1. Copy the value from the **Tenant ID** field. This is your **Tenant ID**.
1. Next you need your **Application ID**:
   - If you created an AAD registered application, navigate to **Azure Active Directory**, **App Registrations**, click **View all applications**, select the app and copy the **Application ID**.  Please note, the Azure UI defaults to **Owned applications** tab.  Click the **All applications** tab to view all app registrations. 
   - If you haven't created a registered app, navigate to **Azure Active Directory**, **App registrations**, click on **New registration** and add the details for your app, and click **Save**. Make note of the **Application ID**.
1. Generate a one-time password by navigating to **Certificates & Secrets** then **New client secret**. Add a new secret, enter a description, and click **Save**. Make note of the displayed application password for use in Octopus. You can change the expiry date, if you don’t want to accept the default one-year expiry for the password.

You now have the following:

- **Tenant ID**
- **Application ID**
- **Application Password/secret**


Next, you need to configure your resource permissions.

### Resource permissions {#resource-permissions}

Resource permissions ensure your registered app has permission to work with your Azure resources.

1. In the Azure Portal navigate to **Resource groups** and select the resource group(s) that you want the registered app to access. If a resource group doesn't exist, create one by going to **Home**, then **Resource groups** and selecting **Create**. After it's created, take note of the Azure subscription ID of the resource group.
2. Click the **Access control (IAM)** option. Under **Role assignments**, if your app isn't listed, click **Add role assignment**. Select the appropriate role (**Contributor** is a common option) and search for your new application name. Select it from the search results and then click **Save**.

The next step is setting up an Azure web application and configuring its properties.

### Web application set up {#web-application-set-up}


1. In your **Resource group** click **Create**, then **Kubernetes Service**
2. Give the cluster a name and select an appropriate region 
3. Click **Review + create**
4. The cluster name will be the AKS cluster name in Octopus Deploy - make note of the resource group name

### Add the service principal account in Octopus {#add-service-principal-account}

With the following values, you can add your account to Octopus:

- Application ID
- Tenant ID
- Application Password/Key

1. Navigate to **Infrastructure**, then select **Accounts**
2. Select **ADD ACCOUNT**, then click **Azure Subscription**
3. Give the account the name you want it to be known by in Octopus
4. Give the account a description
5. Add your Azure Subscription ID - found in the Azure portal under **Subscriptions**
6. Add the **Application ID**, the **Tenant ID**, and the **Application Password/Keyword**

Click **SAVE AND TEST** to confirm the account can interact with Azure. Octopus will attempt to use the account credentials to access the Azure Resource Management (ARM) API and list the Resource Groups in that subscription. 

You may need to safelist the IP addresses for the Azure Data Center that you're targeting. See [deploying to Azure via a firewall](https://octopus.com/docs/deployments/azure/deploying-to-azure-via-a-firewall) for more details.

:::hint
A newly created service principal can take several minutes before the credential test passes. If you've double-checked your credential values, wait 15 minutes and try again.
:::

Next, set up Octopus Deploy to load the YAML file to set up a Kubernetes cluster.

## Octopus Deploy set up

Create a project with a production environment in your Octopus Deploy instance. 

To do this, go to **Infrastructure**, then **Environments**, then **Add Environments** to add the production environment. 
Then, go to **Projects**, **Add Project** to add a project.

Go to **Library**, then **External Feeds** and set up a docker registry. Since we're using the public repository, you can leave **Credentials** blank.

![Docker registry](docker-registry.png "Docker registry")

Set up the Kubernetes target by going to **Infrastructure Deployment Targets**, then **Add Deployment Target**, then **Kubernetes Cluster**. 

Fill out the step:

- **Environments** - The environment you set up in Octopus
- **Target Roles** - `kube`
- **Authentication** - `Azure Service Principal`
   - **Select Account** - The Azure account you set up in Octopus
   - **AKS cluster name** - The name of your Kubernetes cluster
   - **AKS resource group name** - The name of your Azure resource group
   - **Login with administrator credentials** - Check this box

Leave everything else as default and click **SAVE** to finish.

![Kubernetes target](octopus-kubernetes-target.png "Kubernetes target")

In your new project, create a **Deploy Kubernetes** container step by going to **Process**, then **Add Step**, then **Kubernetes**, and then select **Deploy Kubernetes Containers**. 

![Kubernetes deployment step](add-kubernetes-deployment-step.png "Kubernetes deployment step")

Make sure to add the `kube` role under the **on behalf of** option to trigger the build for the Kubernetes deployment target. Paste the YAML file from the K8s YAML tool into the **Edit YAML** section. Leave everything else as default and click **SAVE** to finish.

![Edit YAML](edit-yaml.png "Edit YAML")

Click **Create a release**, and then click the deploy steps to deploy the release. Wait for the success message. 

After the deployment is successful, access the web application by exposing the cluster to the internet. Go to the Azure portal and bring up the PowerShell Azure CLI.

![Azure CLI](azure-cli.png "Azure CLI")

    az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
    
This command will point the CLI to your cluster:

    kubectl get deployments

Running this command will get the list of deployments on the cluster. You should see the deployment `octopus-deployment`. Use this name to expose the web application:

    kubectl expose deployment octopus-deployment --type=LoadBalancer --name=my-service
    
This command creates a service named 'my-service' that generates a public IP to view the web application:

    kubectl get services

Run this command, and you will see "pending" under the External-IP. Wait one minute, run again, and you should see a public IP in that field. Go to the IP address in the browser to view your web application.

![RandomQuotes](random-quotes.png "RandomQuotes")

## Conclusion

In this post, you learned about the new Kubernetes YAML tool with live updates and two-way sync. You can use the tool to generate a YAML file that is compatible with Kubernetes. 

You also ran through a simple use case with Octopus Deploy and Azure Kubernetes Service. You deployed a web application using the YAML file generated by K8s YAML.

Happy deployments!
