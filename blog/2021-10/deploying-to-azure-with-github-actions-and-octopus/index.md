---
title: Deploying to Azure with Github actions and Octopus Deploy
description: Learn how to deploy an Azure web application with Github actions and Octopus Deploy
author: terence.wong@octopus.com
visibility: public
published: 2021-08-02-1400
metaImage: deploy-flow.png
bannerImage: deploy-flow.png
bannerImageAlt: A flow to continuously deploy a web app
isFeatured: false
tags:
 - DevOps
---

This blog post will use Octopus Deploy, Github actions, and Docker to deploy a sample web application to Azure. The application will update on new code changes. To complete the steps, you will need:

- an active Octopus Deploy instance
- a Github account
- a Docker Hub account
- an Azure account

The deployment flow begins with Github. Github hosts the web application code. Github Actions automatically detects changes to the code base, builds the code, and deploys a Docker image to Docker Hub. Octopus Deploy uses this image in an orchestration step to deploy the web application to Azure.

![Deploy Flow](deploy-flow.png "Deploy Flow")

First, let's fork [this repository](https://github.com/OctopusSamples/RandomQuotes-JS). This web application generates random historical quotes on a button press.

![Random Quotes fork](random-quotes-fork.png "Random Quotes fork")

Next, we need to set up Github actions to automate the build, push and deploy process. To do this, we need to retrieve some credentials from Docker and Octopus.

Go to **{{Docker Hub account, account settings, security}}** and create a new access token. Save this token as you can only view it once.

![Docker Token](docker-token.png "Docker Token")

Go to your Octopus instance, then **{{profile,my API keys}}** and create an API key. Save this key value. Take note of your Octopus server URL.

![Octopus API key](octopus-api-key.png "Octopus API key")

In the Random Quotes repository that you forked, go to **{{settings, Secret}}** and add the following repository secrets:

    DOCKER_HUB_ACCESS_TOKEN
    DOCKER_HUB_USERNAME
    OCTOPUS_APIKEY
    OCTOPUS_SERVER

Navigate to .github/workflows where you will see a node.yml file. This file gives instructions to Github on how to deploy the code. Replace the contents of the file with the following code:

    name: deploytoazure

    on:
      push:
        branches: [ master ]
    jobs:

      build:
        name: Build
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v2

          - name: Check Out Repo 
            uses: actions/checkout@v2

          - name: Login to Docker Hub
            uses: docker/login-action@v1
            with:
              username: ${{ secrets.DOCKER_HUB_USERNAME }}
              password: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}

          - name: Set up Docker Buildx
            id: buildx
            uses: docker/setup-buildx-action@v1

          - name: Build and push
            id: docker_build
            uses: docker/build-push-action@v2
            with:
              context: ./
              file: ./Dockerfile
              push: true
              tags: ${{ secrets.DOCKER_HUB_USERNAME }}/randomquotes-js:latest


This code builds and pushes the code as a docker image to Docker Hub on every new push to master. Go to the Github actions tab to view the steps. 

![Github Success Initial](github-success-initial.png "Github Success Initial")

After the build is complete, navigate to [Docker Hub](https://hub.docker.com/) to see the image.

## Configure an Azure Account

We need to configure an Azure account and web application to act as a target for the deployment from Octopus. Other targets are possible such as AWS or local.

Next, you'll create an account in Azure, by navigating to the [portal](https://portal.azure.com/). 

### Create an Azure Service Principal with the Azure Portal {#create-service-principal-account-in-azure}

<iframe width="560" height="315" src="https://www.youtube.com/embed/QDwDi17Dkfs" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

1. In the Azure Portal, open the menu, ![](menu.png) and navigate to **{{Azure Active Directory,Properties}}** and copy the value from the **Tenant ID** field, this is your **Tenant ID**.
1. Next you need your **Application ID**.
 - If you've created an AAD registered application, navigate to **{{Azure Active Directory,App Registrations}}**, click **View all applications**, select the app and copy the **Application ID**. Please note, the Azure UI defaults to **Owned Applications** tab. Click the **All Applications** tab to view all app registrations. 
 - If you haven't created a registered app, navigate to **{{Azure Active Directory,App Registrations}}**, click on **New registration** and add the details for your app, and click **Save**. Make note of the **Application ID**.
1. Generate a one-time password by navigating to **{{Certificates & Secrets,Certificates & Secrets}}**. Add a new **secret**, enter a description, and click **Save**. Make note of the displayed application password for use in Octopus. If you don’t want to accept the default one year expiry for the password, you can change the expiry date.

You now have the following:

- **Tenant ID**
- **Application ID**
- **Application Password/secret**

Now, you can [add the Service Principal Account in Octopus](#add-service-principal-account).

Next, you need to configure your [resource permissions](#resource-permissions).

### Resource permissions {#resource-permissions}

Resource permissions ensure your registered app has permission to work with your Azure resources.

1. In the Azure Portal navigate to **Resource groups** and select the resource group(s) that you want the registered app to access.
2. Next, click the **Access Control (IAM)** option. Under **Role assignments**, if your app isn't listed, click **Add role assignment**. Select the appropriate role (**Contributor** is a common option) and search for your new application name. Select it from the search results and then click **Save**.

Next, you will set up an [Azure web application](#web-application-setup) and configure its properties.

### Web application setup {#web-application-setup}

1. If a resource group doesn't exist, create one by going to **{{Home,Resource groups, Create}}. When created, take note of the Azure subscription ID of the resource group.
2. In your **Resource group** click **{{Create, Web App}}**
3. For the publish setting, choose Docker container.
4. For the operating system, choose linux.
5. Take note of your Azure app name. This will be the address of your web application: [webapp-name].azurewebsites.net
6. Take note of the app service plan and resource group when setting up the application.

In your Octopus Deploy instance, go to **{{Library, External feeds}}** and add the docker container registry feed by entering your docker credentials. Click save and test to confirm the connection.

### Add the Service Principal account in Octopus {#add-service-principal-account}

Now that you have the following values, you can add your account to Octopus:

- Application ID
- Tenant ID
- Application Password/Key

1. Navigate to **{{Infrastructure,Account}}**.
2. Select **{{ADD ACCOUNT,Azure Subscriptions}}**.
3. Give the account the name you want it to be known by in Octopus.
4. Give the account a description.
5. Add your Azure Subscription ID. This is found in the Azure portal under **Subscriptions**.
6. Add the **Application ID**, the **Tenant ID**, and the **Application Password/Keyword**.

Click **SAVE AND TEST** to confirm the account can interact with Azure. Octopus will then attempt to use the account credentials to access the Azure Resource Management (ARM) API and list the Resource Groups in that subscription. You may need to whitelist the IP Addresses for the Azure Data Center you are targeting. See [deploying to Azure via a Firewall](https://octopus.com/docs/deployments/azure) for more details.

:::hint
A newly created Service Principal may take several minutes before the credential test passes. If you have double checked your credential values, wait 15 minutes and try again.
:::

## Adding deployment targets

With Octopus, you can deploy software to Windows servers, Linux servers, Microsoft Azure, AWS, Kubernetes clusters, cloud regions, or an offline package drop. Regardless of where you're deploying your software, these machines and services are known as your deployment targets. Octopus organizes your deployment targets (the VMs, servers, and services where you deploy your software) into environments.

1. Go to **{{Infrastructure, Deployment Targets}}**
2. Select an Azure Web App
3. Enter a Display Name
4. Fill out the environment and target roles
5. Select the Azure Account and Web App created earlier

## Create the project environment

Create a project by navigating to **{{Projects, Add Project}}**. These steps assume a project named 'docker'. 

Add an environment named 'Production' by going to **{{Infrastructure,Environments,Add Environment}}**. 

Navigate to the created project. Under variables, add the following variables with their values:

- app-service-plan
- resource-group
- webapp-name

In the process step, add an Azure script step.

![Azure script](azure-script.png "Azure script")

Add your Azure account configured earlier and copy the following code into the script step:

     $pi = $OctopusParameters["Octopus.Action.Package[randomquotes-js].PackageId"]
     $pv = $OctopusParameters["Octopus.Action.Package[randomquotes-js].PackageVersion"]
     $dockerImage = "${pi}:${pv}"
     $ImageName = $OctopusParameters["Octopus.Action.Package[randomquotes-js].Image"]
     $RegistryUrl = $OctopusParameters["Octopus.Action.Package[randomquotes-js].Registry"]

     az webapp create --resource-group $OctopusParameters["resource-group"] --plan $OctopusParameters["app-service-plan"] --name $OctopusParameters["webapp-name"] --deployment-container-image-name $dockerImage

     az webapp config container set --name $OctopusParameters["webapp-name"] --resource-group $OctopusParameters["resource-group"] --docker-custom-image-name $ImageName --docker-registry-server-url $RegistryUrl

Add a referenced package under the script step by navigating to the docker feed and searching for the random quotes package. Check 'the package will not be acquired' to reference the package in the script. Click OK. Click Save to save the process step.

![Referenced package](referenced-package.png "Referenced package")

We want to ask Github actions to automatically build the docker image, push it to Docker Hub, create a release, and deploy it in the Github code. The update will happen on every commit to master. Update node.yml to the following:

    name: deploytoazure

    on:
      push:
        branches: [ master ]
    jobs:

      build:
        name: Build
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v2
          
          - name: Install Octopus CLI
            uses: OctopusDeploy/install-octopus-cli-action@v1.1.1
            with:
              version: latest
              
          - name: Check Out Repo 
            uses: actions/checkout@v2

          - name: Login to Docker Hub
            uses: docker/login-action@v1
            with:
              username: ${{ secrets.DOCKER_HUB_USERNAME }}
              password: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}

          - name: Set up Docker Buildx
            id: buildx
            uses: docker/setup-buildx-action@v1

          - name: Build and push
            id: docker_build
            uses: docker/build-push-action@v2
            with:
              context: ./
              file: ./Dockerfile
              push: true
              tags: ${{ secrets.DOCKER_HUB_USERNAME }}/randomquotes-js:latest

          - name: sleep
            run: sleep 60
          
          - name: create Octopus release
            run: octo create-release --project docker --version 0.0.i --server=${{ secrets.OCTOPUS_SERVER }} --apiKey=${{ secrets.OCTOPUS_APIKEY }}
            
          - name: deploy Octopus release
            run: octo deploy-release --project docker --version=latest --deployto Production --server=${{ secrets.OCTOPUS_SERVER }} --apiKey=${{ secrets.OCTOPUS_APIKEY }}

The changes have installed the Octopus Deploy CLI onto the machine to run commands on behalf of your Octopus Deploy instance. On every push to Docker, the script waits 60 seconds and then creates a new deployment for Azure. Commit the changes and navigate to the actions tab to confirm the deployments.

![Github success](github-success.png "Github success")

Navigate to Octopus Deploy **{{Projects,Releases}}** to see the latest deployments

![Octopus success](octopus-success.png "Octopus success")

Go to [webapp-name].azurewebsites.net to see your web application:

![Random Quotes 2018](random-quotes-2018.png "Random Quotes 2018")

Let's make a change to confirm that the deployment is automatically updated. In Github, edit the file:

    RandomQuotes-JS/source/www/index.html
 
![Random Quotes year change](random-quotes-year-change.png "Random Quotes year change")

Change the year to 2021 and commit the code to GitHub. The commit and push will trigger the Github actions build. After the deployment is complete, navigate to your web app where the year has changed.

![Random Quotes 2021](random-quotes-2021.png "Random Quotes 2021")

This tutorial set up a continuous delivery flow with Octopus Deploy, Github Actions, Docker, and Azure. Github automatically detects changes to the code and triggers a build and push to Docker. Octopus Deploy then creates a new release and deploys the Azure Web Application. 

Happy deployments!

