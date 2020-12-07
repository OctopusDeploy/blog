---
title: "Infrastructure as code in Azure with Octopus Deploy and Pulumi: Part two"
description: In part one of the blog post series for Pulumi and Octopus Deploy, you learned how to configure Pulumi. In this post, you learn how to tie it together with Octopus Deploy.
author: michael.levan@octopus.com
visibility: private
published: 2199-10-10 
metaImage: 
bannerImage: 
tags:
 - Automation
 - DevOps
---

In [part one](/blog/2020-12/infrastructure-development-in-azure-with-octopus-deploy-and-pulumi-part-1/index.md) of this infrastructure development with Pulumi and Octopus Deploy series, I showed you how to configure Pulumi with a new project and write code with the Pulumi SDK that specifies Azure using Go.

In this post, I go over the deployment aspects and show you how to package and deploy the Go code from part one with Octopus Deploy.

## Using a GitHub repo

There are a few different scenarios that can be used to build and package an application:

- A build server
- Pulling a package from an artifact repo
- Zipping up a package
- Many others...

Figuring out which one to use could be an entire blog posts in itself. For this blog post, we're using GitHub, which covers a free and common scenario.

### Creating a GitHub Repo

First, create a new GitHub repo.

1. Log into [github.com](https://www.github.com).
2. Under repositories, click **New**.
3. Give the repository a name, for example, this blog post will use the name `pulumi-azure-resource-group`.
4. Because this repository doesn't have any sensitive information, it's okay to keep it `public`.
5. Once complete, click **Create repository**:

### Pulling down the repo locally and pushing the code

1. Clone the GitHub repo and copy the Pulumi project that you created in part one into the repo.
2. Commit and push the code to the GitHub repo.

![](images/2.png)

### Create a version

For Octopus Deploy to pull in the external feed from GitHub, the GitHub repo needs a release that's built from the code inside the repo.

1. In the repo, under **Releases**, click **Create a new release**. 

![](images/3.png)

3. Give the release a name and a version number, then click **Publish release**.

A release will be created.

## Configuring an Octopus project for Pulumi

The code is now written, pushed to GitHub, and it is ready to be packaged for deployment using Octopus Deploy. To do this, you need to create a new project and use the Pulumi community step template.

### Creating an external feed

1. Open a web browser and log into the Octopus Deploy Web Portal.
2. Go to **{{Library, External Feeds}}**.
3. Click **ADD FEED**.
4. Under **Feed Type**, choose **GitHub Repository Feed.**
5. For the name, give it the name *Pulumi Azure Resource Group.*

You don't have to add any credentials as the repo is public.

### Creating a new project and project group

1. Navigate to **Projects**.
2. Click **ADD GROUP**.
3. Name the group **Pulumi**.
4. Within the new group, add a new project called **Golang-Pulumi**.

### Creating project variables

For the Pulumi step template to deploy to Azure, it needs authentication for Azure. You can satisfy this requirement by using a project variable of type Azure account.

1. Navigate to the project you created and under **Variables**, click **Project**.
2. Create a new variable of account type **Azure Account.** Ensure that the variable name is **Azure** because the step template searches the project variables for a variable name of **Azure**.
3. Select the Azure account that has access to deploy resources.
4. Save the variable.

### Adding the deployment steps

In the process, there are two deployment steps we will use:

- Deploy a Package
- Run Pulumi (Linux)

In the project you created, go to **Process** to start adding in the steps.

1. The first step is Deploy a Package. Under the Deploy a Package step, specify the name, target roles for the deployment target, and package details. Ensure that the package details are pointing to the GitHub feed and the repo where you stored the Pulumi Azure project.

	Customize the features used by the step:

	- Scroll to the top of the step and click the **CONFIGURE FEATURES** button.
	- Select the **Custom Installation Directory** option.
	- Uncheck the **.NET Configuration Variables** and **.NET Configuration Transforms** options.

2. For the custom installation directory, choose where you want the Pulumi Azure code to reside. This is also where the Pulumi step will look to create the Azure resource group.
3. Save the step.
4. The second step is Run Pulumi (Linux). Under Run Pulumi (Linux), specify the name and target roles for the deployment target. 
5. There are a few parameters under the Run Pulumi (Linux) step template that are specific to the step. Let's go over them:
    - **Stack Name**: The stack name is the full name of the project in Pulumi. It's `OrganizationName/ProjectName/StackName`. For example, mine is `AdminTurnedDevOps/azure-go-new-resource-group/dev`. You can find this information for your stack name in the Pulumi portal.
    - **Create Stack**: This option is only needed if you create a stack. Because a stack already exists, you don't have to worry about using this option.
    - **Command**: Pulumi has several commands, but the one you're interested in is `pulumi up`. As you can see, you don't have to type the full command, just type `up`
    - **Command Args**: The command arg `--yes` is required. When you run Pulumi, say, from the command-line, there's an option that you need to choose to create a resource. The two options are `yes` or `no`. Because we don't have those pop-ups in the step template, we use the `--yes` flag.
    - **Pulumi Access Token**: This is an API key that you can generate in the Pulumi portal under settings.
    - **Pulumi Working Directory**: The working directory is where you copied the code over to from the **Deploy a Package** step.
    - **Restore Dependencies**: This step is for use with NodeJS to restore dependencies. Because you aren't using NodeJS, you can uncheck the box.

6. When complete, save the step.

## Deploying the code

Now it's time to create a new release and run the continuous deployment process to create a new Azure Resource Group using Pulumi and Octopus Deploy.

1. In the Octopus Deploy project, click **CREATE RELEASE**.
2. Click **SAVE**.
3. Deploy to the environment of your choosing, by clicking **DEPLOY TO** and selecting the environment. 
4. Click **DEPLOY**.

You have successfully created an Azure Resource Group using Octopus Deploy and Pulumi.

## Conclusion

When you're taking on the challenge of automating the creation of resources and services, it's no easy task. In fact, nothing is easy when you first start it. However, things become simpler. Combining tools like Octopus Deploy and Pulumi allow you to automate an entire workflow from start to finish so you don't have to worry about manual processes anymore.
