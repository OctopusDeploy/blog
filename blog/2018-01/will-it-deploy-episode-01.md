---
title: Will it Deploy? ASP.NET Core web app to Azure w/ infrastructure provisioning and zero downtime
description: We're try to automate the deployment of a ASP.NET core web app to Microsoft's Azure platform with Infrastructure provisioning and zero production downtime.
author: rob.pearson@octopus.com
visibility: private
published: 2018-01-30
#metaImage: will-it-deploy-meta.png
#bannerImage: will-it-deploy.png

tags:
 - will-it-deploy
---

Today, we're launching 'Will it Deploy?'! This is our brand new video series where we try to automate the deployment of different technologies with Octopus Deploy. 

We're kicking off with a fun one as we try to deploy a ASP.NET Core web app to Microsoft's Azure platform. That alone is pretty easy so we decided to make it a bit more interesting by adding that it should automate the provisioning of the required cloud infrastructure as well as ensure we have a zero-downtime production deployment. Our app is called [Random Quotes](https://github.com/OctopusSamples/RandomQuotes) and it simply gives you a random quote. This is fairly simple but it'll allow us to walk through how to automate the deployment of a web application to Microsoft Azure platform.

<iframe width="560" height="315" src="https://www.youtube.com/embed/Z77T3SHRLKE" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

### Problem

## Tech Stack

* Microsoft [ASP.NET Core 2.0](https://docs.microsoft.com/en-us/aspnet/core/) web app
* [NUnit](http://nunit.org/) unit testing framework

## Deployment Target: 

TODO: Add Azure logo

* Microsoft's Azure Platform - [App Service]
* Provision our cloud infrastructure with an Azure Resource Manager Template (ARM Template)
* Zero-downtime production deploy (Applying the blue-green deployment pattern)

### Solution

So will it deploy? Yes it can! Our deployment process looks like the following.

TODO: Screenshot

We add an Octopus - Azure account so that you can safely and securely deploy 

- Azure web app step
- Azure powershell step

This episode's [GitHub repo](https://github.com/OctopusSamples/WillItDeploy-Episode001) contains all the resources and links used in this video.

### Wrap-up: 

We hope you like this new series and we hope it can help people around the world learn how to automate the depoyment of their application and services.  Don't forget to subscribe to our [YouTube](https://youtube.com/octopusdeploy) channel as we're adding new videos regularly. If there's a framework or technology you'd like us to explore, let us know in the comments.

Happy deployments! :)