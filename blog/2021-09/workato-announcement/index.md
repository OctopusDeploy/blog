---
title: "Workato connector for Octopus Deploy"
description: A Workato connector that integrates with Octopus Deploy is now available.
author: andreia.virmond@octopus.com
visibility: private
published: 2022-09-01-1400
bannerImage: 
metaImage: 
bannerImageAlt: 
isFeatured: false
tags:
- xxx
- xxxx
---

A [Workato connector](https://www.workato.com/integrations) that integrates with Octopus Deploy is now available. Workato is an Integration-Platform-as-a-Service (iPaaS) that integrates apps and automates business workflows. Its connectors contain the building blocks for recipes; including methods of authentication, triggers, and actions for a specific app. 

The Octopus Deploy connector now enables Workato users to:
Quickly integrate Octopus Deploy as part of their workflows. 
Perform operations against Octopus Deploy, such as:

- Add Build Information
- Create Release
- Deploy Release
- Get Deployment Process
- Get Spaces
- Get Projects
- Get Spaces
- Promote Release
- Push Package
- Run Runbook
- Create recipes that respond to events in Octopus through a trigger.

Additional actions may be added to the connector if/when required. 

Built in Ruby, Workato connectors are defined as a JSON configuration and registered through the Workato development environment.

According to Workato, while various teams within IT still play a significant role in [workflow automations](https://www.workato.com/the-connector/work-automation-index/), product teams and business operations teams are getting more and more involved in the process, requiring quick implementations, which can be optimised with Workato’s low-code interface.

## How to get started with Workato

You can find Octopus in the [Workato Integrations](https://www.workato.com/integrations) page, or in the Community Library>Community connectors as a logged-in user.

Below we show a simple example of an integration between Slack and Octopus Deploy using the Octopus Deploy Workato connector:


EXAMPLE ONLY TEXT BELOW

![Project dashboard](individual.png "width=500")

To promote a set of microservices with known versions and in a predictable order, a project can use the [Deploy a release step](https://octopus.com/blog/deploy-release-step/deploy-release-step). By treating the deployments of other projects as deployable resources, the **Deploy a release** step allows teams to capture the state of an environment at a given point in time, and deploy that state to the next environment.

In the screenshot below you can see an example of an Octopus project that includes a sequence of **Deploy a release** steps. The order of these steps ensures that the microservices are deployed in a fixed order, and the set of **Deploy a release** steps represents a complete microservice ecosystem that is deployed as a single unit to new environments. 

You can see this project configured on our [demo server](https://demo.octopus.com/app#/projects/coordinate-deployment/process).



## Conclusion

XXXXX

Happy deployments!
