---
title: Deploying an Java Spring Boot web app - Will it Deploy? Episode 2
description: We try to automate the deployment of a Java Spring Boot web app AWS Elastic Beanstalk with infrastructure provisioning and zero production downtime.
author: rob.pearson@octopus.com
visibility: private
published: 2018-02-08
metaImage: metaimage-will-it-deploy.png
bannerImage: blogimage-will-it-deploy.png
tags:
 - Will it Deploy
---

Wecome to another **Will it Deploy?** Episode where we try to automate the deployment of different technologies with Octopus Deploy.  In this episode, we're trying to deploy an Java Spring Boot web app to Amazon Web Services platform with cloud infrastructure provisioning as well as ensure we have a zero-downtime production deployment.

<iframe width="560" height="315" src="https://www.youtube.com/embed/TODO" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

## Problem

### Tech Stack

Our app is a quote generator called [Random Quotes](https://github.com/OctopusSamples/WillItDeploy-Episode002). The application is pretty simple but it'll allow us to illustrate how to go deploy a Java web app to Amazon Web Services platform.

* [Spring Boot](https://projects.spring.io/spring-boot/) web app.
* [JUnit](http://junit.org/) unit testing framework, Mokito and Hamcrest.

Kudos to our marketing manager [Andrew](https://twitter.com/andrewmaherbne) who has been learning to code and built the first cut of this app. Great work! 

### Deployment Target

![Microsoft Azure logo](will-it-deploy-azure-logo.png "width=500")

* AWS - [Elastic Beanstalk](https://aws.amazon.com/elasticbeanstalk/).
* Provision our cloud infrastructure with an AWS [CloudFormation Template](https://aws.amazon.com/cloudformation/).
* Zero-downtime production deploy - [applying the blue-green deployment pattern](https://octopus.com/docs/deployment-patterns/blue-green-deployments).

## Solution

So will it deploy? **Yes it will!** Our deployment process looks like the following.

TODO ![Octopus deployment process](will-it-deploy-deployment-process.png "width=500")

The first step is to add an Octopus AWS account, which has all the details required to enable me to connect to the AWS platform, safely and securely. It is used to authenticate with AWS when deploying or executing scripts.

![Octopus AWS account](will-it-deploy-aws-account.png "width=500")

Then we add the following steps to successfully deploy our app including cloud infrastructure provisioning and a zero downtime production deployment.

TODO:

- Octopus **Deploy an Azure Resource Group** step to provision our cloud infrastructure via an ARM Template.
- Octopus **Run an Azure Powershell Script** step to ensure we always have a fresh App Service staging deployment slot. We call the Azure Powershell cmdlets to delete and create an App Service deployment slot.
- Octopus **Deploy an Azure Web App** step to deploy our web application to our App Service staging deployment slot.
- Octopus **Run an Azure Powershell Script** step to swap our App Service staging and production (live) deployment slot. This is only done during a production deployment so that we achieve zero-downtime!

This project uses the following variables to store our resource group name, website name, and app settings. Nice and simple!

![Project variables](will-it-deploy-project-variables.png "width=500")

This episode's [GitHub repo](https://github.com/OctopusSamples/WillItDeploy-Episode001) contains all the resources and links used in this video.

### Wrap-up

We hope you enjoyed this episode as we have many more in the works! If there's a framework or technology you'd like us to explore, let us know in the comments.

Don't forget to subscribe to our [YouTube](https://youtube.com/octopusdeploy) channel as we're adding new videos regularly. Happy deployments! :)

