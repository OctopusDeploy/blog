---
title: Opsera Announces Integration with Octopus Deploy
description:
author: john.bristowe@octopus.com
visibility: public
published: 2022-03-14-1400
metaImage:
bannerImage:
bannerImageAlt:
isFeatured: false
tags:
  - DevOps
  - CI Series
  - Continuous Integration
  - Partners
---

I'm excited to share that we've partnered with [Opsera](https://www.opsera.io/) to provide integration with Octopus Deploy. This enables you to integrate Octopus Deploy with the 95+ platforms that Opsera supports as part of its no-code DevOps orchestration platform.

t can be a challenge to automate your CI/CD pipeline, especially when it's composed of disparate solutions. It requires technical know-how and experience to orchestrate the various tools, pipelines, and insights required by your development teams. This is where Opsera can help. Opsera provides a self-service platform to help you automate your CI/CD through a visual interface, enabling you to create declarative pipelines with unified insights.

Once enabled, Opsera's integration provides access through its pipeline UI to resources defined in Octopus Deploy such as channels, environments, lifecycles, projects, spaces and tenants.

## Getting Started

Communication between Opsera and Octopus Deploy is performed through the Octopus API. Therefore, before configuring Octopus Deploy in the Tool Registry for Opsera, you need to create an API key in Octopus Deploy. This API key is used to perform operations against Octopus Deploy on your behalf. Information on how to do this is provided in our [How to Create an API Key documentation](https://octopus.com/docs/octopus-rest-api/how-to-create-an-api-key).

The next step is to register Octopus Deploy in Opsera through the Tool Registry. Instructions for this step can be found in the section entitled, [Registering the Octopus Tool in Tool Registry](https://opsera.atlassian.net/wiki/spaces/OE/pages/1367474335/Octopus+Deployment#Registering-the-Octopus-tool-in-Tool-Registry) in the documentation provided by Opsera. Once this step is completed, you’ll be able to use the Octopus Deploy integration in the pipeline UI of Opsera.

## An Overview of Opsera Integration with Octopus Deploy

A powerful feature of Opsera is its ability to visualize and orchestrate pipeline workflows:

![Workflow Visualization](workflow.png)

This workflow represents a Java-based project: build source code through the command line → push artifacts to Nexus repository → deploy to IIS through Octopus Deploy. Opsera displays each step of a workflow; each step represents a tool and an operation to be performed.

![](step-setup.png)

Editing the configuration enables you to customize the properties and actions encompassed by its step:

![](octopus-integration-settings.png)

This step represents a deployment to IIS through Octopus Deploy. Resources like channels, projects, and roles are exposed through the Opsera UI, providing a consistent experience across the platforms it supports.

Once the pipeline has been established, it can be run based on a series of events or as a scheduled task:

![](pipeline-logs.png)

## Conclusion

Opsera provides a self-service platform to help you automate your CI/CD through a visual interface, enabling you to create declarative pipelines with unified insights.

Octopus Deploy joins the 95+ platforms supported by Opsera as part of its no-code DevOps orchestration platform. The power of Opsera is its ability to easily integrate other systems with Octopus Deploy.

Happy deployments!
