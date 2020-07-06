---
title: From CI to CD
description: In this post we link up Jenkins and Octopus to form a CI/CD pipeline
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

In the previous blog post we used Octopus to build a Kubernetes in AWS with the EKS service and then deployed the Docker image created by Jenkins as a Kubernetes deployment and service.

However, we still don't have a complete CI/CD solution, as Jenkins is not integrated with Octopus, leaving us to manually coordinate builds and deployments.

In this blog post we'll extend our Jenkins build to call Octopus and initiate a deployment once our Docker image has been pushed to Docker Hub.

## Installing the Jenkins plugins

Octopus provides a plugin for Jenkins that exposes integration steps both in freestyle projects and in pipeline scripts. This plugin can be installed via {{ Manage Jenkins, Manage Plugins }}:

![](octopusplugin.png "width=500")

The Octopus plugin uses the Octopus CLI to execute the actions. We can install the CLI manually on the agent, but for this example we'll use the **Custom Tools** plugin to download the Octopus CLI and push it to the agent:

![](customtoolsplugin.png "width=500")

## Configuring the Octopus server and tools

The details of the Octopus server that our pipeline will connect to is defined under {{ Manage Jenkins, Configure System }}:

![](octopusserver.png "width=500")

We then need to define a custom tool under {{ Manage Jenkins, Global Tool Configuration }}. The custom tool has the name of **OctoCLI**, and because in my case the agent is running on Windows, the Octopus CLI will be downloaded from https://download.octopusdeploy.com/octopus-tools/7.4.1/OctopusTools.7.4.1.win-x64.zip:

![](octocli.png "width=500")