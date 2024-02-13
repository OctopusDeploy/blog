---
title: Octolint best practices
description: Learn the best practices Octolint is based on so you can understand the recommendations and avoid undesirable Octopus configurations.
author: steve.fenton@octopus.com
visibility: public
published: 2024-02-14-1400
metaImage: blogimage-runbooksbestpractices-2022.png
bannerImage: blogimage-runbooksbestpractices-2022.png
bannerImageAlt: A book with legs and arms is wearing a toolbelt and running with a checklist in its hand
isFeatured: false
tags: 
  - Product
---

Octolint is a tool that scans your Octopus instance and recommends improvements to your setup. This post explains the best practices we based Octolint on. It will help you understand the recommendations and avoid undesirable Octopus configurations.

You can create a runbook to perform the Octolint scan using the Octolint step template or call it from the command line using the script on the step template page. You can use Octolint's recommendations to help you get the best from your Octopus instance.

```bash
$server = $OctopusParameters["Octolint.Octopus.ServerUri"]
$apiKey = $OctopusParameters["Octolint.Octopus.ApiKey"]
$spaceName = $OctopusParameters["Octolint.Octopus.SpaceName"]

docker pull octopussamples/octolint

docker run --rm octopussamples/octolint -url "$server" -apiKey "$apiKey" -space "$spaceName" -verboseErrors
```

When you run Octolint in a runbook, the task log will contain messages that provide recommendations, like:

```
OctoRecLifecycleRetention
The following lifecycles have retention policies that keep releases or files forever: 
Default Lifecycle
```

In this post, you'll learn more about what each recommendation means.

## Environment count

> A space should have 10 or fewer environments.

At a high level, environments model business processes and group distinct infrastructure copies.

Environments typically represent copies of infrastructure that deployments progress through as they gain more stability and confidence that changes are ready to be exposed to end users. 

Teams may have special responsibilities for given environments. For example, only product owners can approve production deployments.

Having too many environments is a sign they're being used to model a different concept, like projects, regions, or customer-specific environments.

The tenants feature can represent concepts like customers, users, regions, or physical locations.

## Default project group size

> The default project group should have 10 or fewer projects.

Project groups show the progression of project deployments on the Octopus dashboard. You can also filter the dashboard to show only a subset of project groups.

Project groups also provide a security boundary with user roles that you can scope to a project group.

Organize projects into specific groups.

## Empty projects

> Projects should be deleted when not used.

Every project should either have a deployment process or one or more runbooks. Empty projects likely indicate projects that were never properly configured and that you should delete.

## Project-specific environments

> Environments should be reusable.

Environments typically represent high-level business processes and copies of infrastructure that deployments progress through. Environments used exclusively by one project may indicate an anti-pattern, as this can lead to unnecessarily specific environments, which in turn can be challenging to manage.

Consider whether you can represent your project-specific environment with a more general, shared environment.

## Unused variables

> All variables should be used.

Project-level variables can only be referenced by the project deployment process, runbooks, or other variables. Unused variables can clutter the project settings and make it harder to manage.

:::warning
Octolint can only detect if variables are not referenced by the deployment process or by other variables. There are edge cases Octolint can not detect. For example, if an output variable from a step references the project variable, or if an external package includes files that have variable substitution applied to them.
:::

### Cleaning up unused variables

To clean up unused variables, first rename the variables with a common prefix like `[UNUSED]`. For example, if Octolint reported the variable `ConnectionString` was unused, you would rename it to `[UNUSED] ConnectionString`.

Test that the deployment or runbooks continue to work as expected with the renamed variables. If there are no issues, then you can delete the variable. If it turns out the variable is still used, rename it back.
This process ensures you don't delete the variable before proving it's no longer used.

## Duplicated variables

> Variables should represent unique values.

You'll often have a variable you use in many projects. If you copy and paste the value into each project's variables, it becomes difficult to manage the variable. You'd have to change it multiple times.
Library variable sets are a better solution for sharing variables, as you can update their value centrally once.

## Deployment queued by administrators

> Administrator accounts shouldn't be used for day-to-day operations.

Octopus provides many roles to represent typical responsibilities in a deployment pipeline. Admin users initiating deployments may indicate that the Octopus RBAC configuration does not reflect deployment responsibilities.

This is like using a Windows administrator or Linux root account for day-to-day operations.

Create users with restricted permissions to perform deployments.

## Perpetual API keys

> API keys should automatically expire.

API keys can either have an expiration date, or never expire. Keys that never expire may pose a security risk as they grant anyone with the key perpetual access to the Octopus instance.

Replace perpetual keys with keys that expire.

## Project groups with exclusive environments

> Projects in the same group should have the same environments.

Project groups contain related projects and group them on the dashboard. You can see all environments for all projects in a project group horizontally across the screen. This works well when projects in a project group share environments. However, it's inefficient when projects deploy to different sets of environments.

Move projects into groups with projects that deploy to shared environments.

## Too many steps

> A deployment process should have 20 or fewer steps.

Complex deployments can often involve many steps. However, deployment processes can become brittle and hard to manage when they include too many steps.

You can create runbooks to group steps into a reusable unit or use separate projects to reduce the number of steps in a deployment process.

## Direct tenant references

> Explicit groups should be created to manage tenants.

You'll often group tenants based on shared properties like regions, release rings, performance tiers, etc. These groupings are best expressed as tenant tags.

However, it's also possible to refer to groups of tenants directly, creating an implicit group of tenants. You can see this when the same group of tenants is directly referenced across multiple resources like accounts, certificates, and targets. Replacing these implicit groups of tenants with explicit tenant tags makes it easier to manage the Octopus instance.

Use tags to group tenants and use these tags rather than specific tenants for accounts, certificates, and targets.

## Unhealthy targets

> Targets should not remain unhealthy for long periods.

Targets include health checks that indicate if they're online and healthy. Unhealthy targets are still included in deployments by default but are unlikely to complete the deployment successfully. Targets that have been unhealthy for some time likely need attention.

Resolve health issues with targets, or remove them if they're no longer used.

## Shared Git usernames

> Projects should use unique Git usernames.

Config as Code-enabled projects can either define project-specific Git credentials or reference shared credentials. Projects that define the same Git username likely indicate that they're duplicating Git credentials, which makes the Octopus instance harder to maintain.

Create shared Git credentials to use in place of project-specific Git credentials.

## Deployment queue times

> Deployments should queue for less than a minute.

The Octopus task cap limits how many tasks you execute in parallel. When the task queue exceeds the task cap, tasks wait for their turn to start. If too many tasks take too long to start, deployments and runbooks can be delayed.

Increase the task cap, or if running Octopus HA, add another Server node to spread tasks between instances.

## Unused targets

> Targets should be actively used.

Targets represent where you perform deployments. Targets count towards license limits and often have health check tasks scheduled at regular intervals.

You may not require targets you haven't used in the last month, but could still be creating tasks and contributing to the active target count on your license.

Remove any targets you're no longer using.

## Lifecycle retention

> Retention policies should be used to clean up old resources.

Lifecycle retention rules let you specify that resources, like releases and packages, get retained for a specific time or let you retain the last specified number of resources. You can also configure lifecycles to retain resources indefinitely.

Retaining resources indefinitely is inefficient and can introduce performance issues over time.
Use retention rules to clean up old resources to improve performance.

## Good practices

You don't have to remember all these good practices. Use Octolint to do the hard work so you have more time to implement improvements. 

You can watch an [introduction to Octolint on YouTube](https://www.youtube.com/watch?v=hdJ3kbCPVds) to see it in action and find the code on [GitHub](https://github.com/OctopusSolutionsEngineering/OctopusRecommendationEngine).

Whether you improve your security by replacing API keys that don't expire or use step templates to convert complex processes into reusable templates, Octolint is a good way to find out what you can improve and make sure things stay in good shape.

Happy deployments!
