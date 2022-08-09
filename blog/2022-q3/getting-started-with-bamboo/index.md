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

Agent set-up instructions

## Connect to the underwater application github

github instructions

## Setup a project and plan

add the code checkout step
add the docker build step
add the docker push step

## Deploy step

link to other blogs that go through the underwater app step

for now, run and view through the docker app

screenshot of app

## Learn more

link to CI series

## Conclusion

