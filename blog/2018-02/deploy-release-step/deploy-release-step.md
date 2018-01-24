---
title: "Coordinating Projects with the Deploy Release Step"
description: "We have introduced a new 'Deploy Release' step type which allows coordination between Octopus Projects"
author: michael.richardson@octopus.com
visibility: private
tags:
 - New Releases
 - Patterns 
---

Unless you are building a genuine monolith, your projects don't exist in isolation. More and more our industry is moving in a direction where systems are composed of more granular components. 
Here at Octopus HQ we love this trend, because frankly the more moving parts your release and deployment processes have, the more value Octopus brings to the table. 

As a natural consequence of this, users want to be able to coordinate the release of multiple projects in Octopus. This is one of our top [UserVoice suggestions](https://octopusdeploy.uservoice.com/forums/170787-general/suggestions/9811932-allow-project-dependencies-so-deploying-one-proj).

The two key scenarios are:

- **Bundle:** You may want to create a "bundle" release to allow releases of multiple projects to progress through your environments as a unit. 
- **Dependencies:** You want to explicitly model that Project A depends on Project B having been deployed.

## Introducing the _Deploy a Release_ step 

![Deploy Release Step Card](deploy-release-card.png "width=500")

To solve this problem, we have a created a new step: _Deploy a Release_.  The _Deploy a Release_ step allows you to select another Octopus project to deploy.

When you create a release of a project containing one or more _Deploy a Release_ steps, you can select the release versions of the child projects to be deployed.  Exactly as versions of packages are selected when creating a release of projects which contain steps which deploy packages.

The nice thing about implementing this as a step is all the regular Octopus goodness works as expected. You can intersperse _Deploy a Release_ steps with other step types.  For example, if you are creating a bundle project then your first step may be a _Manual Intervention_ step (to approve the release), and your final step may be to send a Slack notification. _Deploy a Release_ steps can also be configured to run only for specific environments, channels, or tenants, just as any other step can.    

When a _Deploy a Release_ step is run, it triggers a deployment of the specified project. This deployment is a no different from a deployment triggered directly.  It will be visible on the Octopus dashboard.   

### Conditional Deployment
You can configure the conditions under which the child project is deployed:

- Deploy Always (default)
- If the selected release is not the current release in the environment
- If the selected release has a higher version than the current release in the environment

### Variables

Variables can be passed to the deployments triggered by a _Deploy a Release_ step. These are available to the child deployment process just as any other project variable. 

Output variables from deployments triggered by a _Deploy a Release_ step are captured and exposed as output variables on the _Deploy a Release_ step. 

This allows output from a child deployment to be used by the parent process and even passed into deployments triggered by subsequent _Deploy a Release_ steps.  Many coordination scenarios are enabled by this. 

## When can I get my hands on it?

This feature will ship with Octopus version 2018.2, which will be released in early February.

_Happy (multi-project) Deployments!_



