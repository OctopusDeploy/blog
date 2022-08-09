---
title: Getting started with bamboo
description: Getting started with bamboo. Learn how to install bamboo, build and push a docker image to a conainer registry.
author: terence.wong@octopus.com
visibility: private
published: 2022-08-15-1400
metaImage: blogimage-top8containerregistries-2022.png
bannerImage: blogimage-top8containerregistries-2022.png
bannerImageAlt: Three-tiered shelf housing eight blue containers.
isFeatured: false
tags: 
  - Continuous Integration
  - Containers
---

Continuous Integration (CI) servers are an important step in the CI/CD process. CI servers take a code repository, build it and push it to a central location where a continuous delivery (CD) tool like Octopus can take over and manage deployments. Bamboo is a CI server developed by Atlassian that can automate the building and testing of software applications.

This blog will take you through:

- Installing Bamboo on a Windows Server
- Configuring a Bamboo project
- Configuring a Bamboo plan 
- Installing and configuring the dependencies required to build and push a docker container to a container registry
- Running and viewing the container image

## Pre-requesites

To follow along with the guide, you will need the following software set up on your Windows Server:

- Java 8 or 11, [according to Atlassian these are the versions they support](https://confluence.atlassian.com/bamboo/supported-platforms-289276764.html)
- [Docker](https://docs.docker.com/desktop/install/windows-install/)
- Git 
- A Github Account
- DockerHub account

## Installing Bamboo on a Windows Server

To try Bamboo, you can [download the latest release](https://www.atlassian.com/software/bamboo/download)

Set the install location to a directory you can access, such as C:\Users\Username\Documents. Setting it to the Default location of C:\Program Files may result in permission errors.

You will also be asked to set the Bamboo home directory, make sure this is a seperate directory from the install location with a folder name Bamboo-home.

Once the install has finished, run the bamboo server:

- Open terminal and navigate to the Bamboo installation directory
- Run bin\start-bamboo.bat
- The server should be started at http://localhost:8085/

## Setting up users

In the start up screen, you will be asked to set up an admin account. Fill out the details and store the details in a password manager.

## Agents

Agents are the workers that execute workloads in Bamboo. Since you have installed the pre-requisite technology, you can use the local machine as an agent for testing purposes.

In the Bamboo dashboard, go to the settings icon and agents

Go to add local agent and give it a name

Click **Save**


## Setup project and plan

In the home menu, click **Create** and **Create Plan**. Fill out the names of your project and plan

![Create Project and Plan](create-project-and-plan.png)

## Connect to the underwater application github

In the next screen check the box that says 'link new repository'

We will be using the [Octopus Underwater App](https://github.com/OctopusSamples/octopus-underwater-app). To use this repository, fork it into you own account. 

In the password settings, use a [personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) to grant Bamboo access to repositories under your GitHub account.

Select the main branch

Test the connection to make sure that Bamboo can connect to this repository

## Configure plan

Leave the isolate build as 'Agent environment'. This will use the local agent you set up earlier.

Bamboo works by tasks. Each task is meant to execute a certain step in the CI pathway, such as checkout, build, pull, push etc.

You will see that there is a source code checkout step pre-filled for you. This checks out the linked Github repository into Bamboo

First, add the build docker task:

- Click on **add Task** and search for 'Docker'
- Set the command to Build a Docker Image
- Set the repository to be [Your DockerHub Username]/[The tag of your image]
- Check **Use an existing Dockerfile located in context path**
- Click **Save**

Now, add the push docker task:

- Click on **add Task** and search for 'Docker'
- Set the command to Build a Docker Image
- Set the repository to be [Your DockerHub Username]/[The tag of your image]
- Check **Use the agent's native credentials**
- Click **Save**

Click **Create**

The plan will start to execute by checking out the code, building the docker image, and pushing the built image to DockerHub

Once done you will see a success green tick box to indicate that the plan has completed successfully.

![Bamboo Success](underwaterapp-success.png)

Navigate to your DockerHub account to confirm that the image has been pushed to the repository.

## Deploy step

Now that the image is on DockerHub, any CD tool can use that to deploy it to locally or to a cloud platform. We have written a guide on how to do this for [Azure](https://octopus.com/blog/deploying-java-app-docker-google-azure), [AWS through GithubActions](https://octopus.com/blog/multi-environment-deployments-github-actions), and [AWS through Jenkins](https://octopus.com/blog/multi-environment-deployments-jenkins).

To view the application locally:

- docker pull [Your DockerHub Username]/[The tag of your image]
- docker run -p 8080:8080 [Your DockerHub Username]/[The tag of your image]
- Go to http://localhost:8080/

You will see the Octopus Underwater app where you can learn more about CI/CD and Octopus

![Octopus Underwater App](octopus-underwater-app.png)

## Learn more

link to CI series

## Conclusion

