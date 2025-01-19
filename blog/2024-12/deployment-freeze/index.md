---
title: More control over Deployment Freezes: Recurring freezes and freeze by tenant
description: We've improved our Deployment Freezes feature, allowing you to create maintenance windows with recurring freezes and have more granular control over the scope of freezes with freezes by tenant.
author: huy.nguyen@octopus.com
visibility: public
published: 2024-12-19-1400
metaImage: blogimage-deploymentfreezes-2024-750x400.png
bannerImage: blogimage-deploymentfreezes-2024-750x400.png
bannerImageAlt: Two people looking at an Octopus Deploy interface showing deployment versions with a winter theme and snowflakes
tags: 
- Product
- Deployment
---

Do you have specific windows where you don't want deployments going out? Do you want to prevent deployments to certain tenants without blocking the whole project?
We've got great news! We're releasing improvements to our Deployment Freezes feature in Octopus Deploy to allow you to do just that.

## Why Do You Need Deployment Freezes?

Let's face it - there are times when deploying new changes is just too risky. Think about:
- Holiday seasons when your team is short-staffed
- Critical business periods like tax season or elections
- Important demos or testing cycles when you need your systems stable
- You need to prevent deployments to production during business hours
- Your customers / branches are modelled with tenants and require more granular control around when deployments to those customers can proceed

Our new feature helps you handle all these cases. It lets you control exactly when and where deployments can happen, keeping your systems stable when it matters most.

## Setting Up Your Freezes

Setting up a deployment freeze is straightforward. You can create them for right now or schedule them for later. You get to choose:
- A clear name so everyone knows what the freeze is for
- When it starts and ends
- How long it lasts
- Which projects, environments, and tenants it affects

Need to set up regular maintenance windows? No problem! You can set up automated recurring freezes that match your schedule.

![Deployment Freeze Detail](deployment-freeze-detail_w.png "width=500")

## Choosing What to Freeze

You can control your freezes by scoping them to projects or tenants.

With project scopes, you can freeze specific projects by environments. For example, you might want to freeze your online store production deployments during Black Friday but keep working on your development and test environments.

![Project Scope](deployment-freeze-project-scope_w.png "width=500")

Tenant scope lets you get even more specific. You can freeze deployments for particular tenants in selected projects and environments. This is helpful if you're managing systems across different time zones or regions.

![Tenant Scope](deployment-freeze-tenant-scope_w.png "width=500")

By mixing these options, you can create freezes that match exactly what your team needs.

## What Happens During a Freeze?

Here's what you can expect when a freeze is active:
- Existing deployments that started before the freeze will continue to completion
- New deployments attempted during the freeze will need to provide an override reason to be able to deploy
- Scheduled deployments that would start during a freeze period will not execute
- Automatic deployments based on deployment target triggers are allowed to ensure your deployment targets stay updated when scaling up
- Other automatic deployments, such as scheduled deployments or automatic lifecycle promotions, are blocked during the freeze

## Setting Up Regular Freezes

Need to set up regular freeze periods? Our recurring schedule feature has got you covered. You can:
- Set up different freezes for each global region
- Match your maintenance windows
- Create daily protection windows

The system lets you set up these recurring patterns:
- Daily windows to protect business hours
- Weekly schedules for regular maintenance
- Monthly patterns for release cycles
- Annual schedules for recurring business events

![Deployment Freeze Recurrence](deployment-freeze-recurrence_w.png "width=500")

## Override Capabilities and Audit Trail

We know that sometimes you need to make exceptions. That's why we've made it easy for authorized users to override active freezes. They just need to provide a reason, which we'll keep track of in the audit trail.

The audit trail keeps a record of:
- Freeze creation and modification
- Deployment Freeze Override and their justifications
- Deployment attempts during freeze periods

![Screenshot of the Deployment Freeze interface](deployment-freeze-override_w.png "width=500")

## Automation and Integration

You can automate and integrate deployment freezes into your workflows using the Octopus REST API or the Terraform provider. For example, you can use the Go client to automate the creation and management of deployment freezes as part of your CI/CD pipeline. The Terraform provider offers an `octopusdeploy_deployment_freeze` resource for creating and managing deployment freezes. For more information, see the following resources:
- [Octopus REST API - Go Client](https://github.com/OctopusDeploy/go-octopusdeploy)
- [Terraform provider: Deployment Freeze example](https://github.com/OctopusDeployLabs/terraform-provider-octopusdeploy/tree/main/examples/resources/octopusdeploy_deployment_freeze)

## Conclusion
Deployment freezes in Octopus Deploy let you control your deployment schedule. This feature helps you protect critical systems during peak times. It helps with reduced staffing and coordinating deployments across teams. It lets you stay agile while maintaining stability. Ready to start with deployment freezes? Check out our [deployment freeze documentation](https://octopus.com/docs/deployments/deployment-freezes) for detailed setup instructions and best practices.

Happy deployments!