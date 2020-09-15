---
title: Using Runbooks To Provision Infrastructure
description: How to use Runbooks to provision/destory infrastructure and give self service access to teams in the organisation.
author: Adam Close
visibility: private
published: 3020-01-01
metaImage: 
bannerImage: 
tags:
  - runbooks
---

By now, you have probably heard the term *infrastructure as code*. Why make manual configuration changes to your infrastructure when you can write a line of code to do it! Iac is great for automating the provisioning of your infrastructure, but if you're in the *infrastructure* team, you're probably still the person that deploys the code to provision the infrastructure. Who's usually requesting new infrastructure? It's the software engineers, right? The engineers have created a feature branch and then want to test the changes on some new servers but have to go to the infrastructure team to request it. Using Octopus Runbooks, you can deploy your Iac code and give self-service access to other teams.  

In this blog post, I'll show you how to create Runbooks to create & destroy application servers. Control cloud expenditure and give your organizations teams the ability to safely spin up infrastructure on-demand in a controlled and gated process. 


### Setup Infrastructure Release Team

Firstly, if we're enabling self service access to server poviosning we want to get some control around when the servers should be provsioned. Using manaual intervention steps in Octopus we can request approval from other teams before Runbooks steps are triggered. The step will require a team to approve the Runbook run so first we need to create a team of octopus users that will be responsable for aproving server provisoning.



### Creating infrastrcutre 

* Create Runbook
* Step 1 Notifcation
* Step 1 Get approval approval 
* Step 2 Deploy Iac Step
* Step 4 Notification 

1. To create a runbook, navigate to **{{Project, Operations, Runbooks, Add Runbook}}**.
2. Give the runbook a name and click **SAVE**.
3. Click **DEFINE YOUR RUNBOOK PROCESS**, then click **ADD STEP**.
4. 
### Destorying infrastucture 


### Destorying infrastucture on a schedule 

Using Runbook to destory infrastcuture is great for saving costs on 

### Setting up teams to run Runbooks 

Lastly, I want to control who can Run the new Runbooks I've created. 

## Conclusion

In this post I showed you how to use Runbooks to provsion servers in the cloud and control cloud costs with triggers. 

## Learn more

- [link](https://www.example.com/resource)
