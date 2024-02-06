---
title: Octolint best practices
description:  This page explains the best practices Octolint is based on so you can understand the recommendations and avoid undesirable Octopus configurations.
author: steve.fenton@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - Product
---

Octolint is a tool that scans your Octopus instance and recommends improvements you can make to your setup. This page explains the best practices Octolint is based on so you can understand the recommendations and avoid undesirable Octopus configurations.

You can create a runbook to perform the Octolint scan using the Octolint step template or call it from the command line using the script on the step template page. You can use Octolint to get recommendations that help you get the best from your Octopus instance.

```bash
$server = $OctopusParameters["Octolint.Octopus.ServerUri"]
$apiKey = $OctopusParameters["Octolint.Octopus.ApiKey"]
$spaceName = $OctopusParameters["Octolint.Octopus.SpaceName"]

docker pull octopussamples/octolint

docker run --rm octopussamples/octolint -url "$server" -apiKey "$apiKey" -space "$spaceName" -verboseErrors
```

When you run Octolint within a runbook, the task log will contain messages that provide recommendations, such as:

```
OctoRecLifecycleRetention
The following lifecycles have retention policies that keep releases or files forever: 
Default Lifecycle
```

In this article, you'll learn more about what each recommendation means.

## Environment count

> A space should have 10 or fewer environments.

At a high level, environments are used to model business processes and group distinct infrastructure copies.

Environments typically represent copies of infrastructure that deployments progress through as they gain more stability and confidence that changes are ready to be exposed to end users. Teams may have special responsibilities for given environments, e.g., only product owners can approve production deployments.
Having too many environments is a sign they are being used to model a different concept, such as projects, regions, or customer-specific environments.

The tenants feature can represent concepts such as customers, users, regions, or physical locations.

## Default project group size

> The default project group should have 10 or fewer projects.

Project groups show the progression of project deployments on the Octopus dashboard. The dashboard can also be filtered to show only a subset of project groups.

Project groups also provide a security boundary with user roles that can be scoped to a project group.
Organize projects into specific groups.

## Empty projects

> Projects should be deleted when not used.

Every project should either have a deployment process or one or more runbooks. Empty projects likely indicate projects that were never properly configured and should be deleted.

## Project-specific environments

> Environments should be reusable.

Environments typically represent high-level business processes and copies of infrastructure that deployments progress through. Environments used exclusively by one project may indicate an antipattern, as this can lead to unnecessarily specific environments being created, which in turn can be challenging to manage.

Consider whether your project-specific environment can be represented with a more general, shared environment.

## Unused variables

> All variables should be used.

Project-level variables can only be referenced by the project deployment process, runbooks, or other variables. Unused variables can clutter the project settings and make it harder to manage.

Note! octolint can only detect if variables are not referenced by the deployment process or by other variables. There are edge cases octolint can not detect, for example if an output variable from a step references the project variable, or if an external package includes files that have variable substitution applied to them.

### Cleaning up unused variables

To clean up unused variables, first rename the variables with a common prefix like [UNUSED]. For example, if octolint reported the variable ConnectionString was unused, you would rename it to [UNUSED] ConnectionString.

Test that the deployment or runbooks continue to work as expected with the renamed variables. If there are no issues, then the variable can be deleted. If it turns out the variable is still used, rename it back.
This process ensures you do not delete the variable before proving its no longer used.

## Duplicated variables

> Variables should represent unique values.

You'll often have a variable that is used in many projects. If you copy and paste the value into each project's variables, it becomes difficult to manage the variable. You'd have to change it multiple times.
Library variable sets are a better solution for sharing variables, as you can update their value centrally once.

## Deployment queued by administrators

> Administrator accounts shouldn't be used for day-to-day operations.

Octopus provides many roles to represent typical responsibilities in a deployment pipeline. Admin users initiating deployments may indicate that the Octopus RBAC configuration does not reflect deployment responsibilities.

This is like a Windows administrator or Linux root account being used for day-to-day operations.
Create users with restricted permissions to perform deployments.

## Perpetual API keys

> API keys should automatically expire.
> 
API keys can either have an expiration date, or never expire. Keys that never expire may pose a security risk as they grant anyone with the key perpetual access to the Octopus instance.
Replace perpetual keys with keys that expire.

## Project groups with exclusive environments

> Projects in the same group should have the same environments.

Project groups contain related projects and group them on the dashboard. All environments for all projects in a project group are shown horizontally across the screen. This works well when projects in a project group share environments but is inefficient when projects deploy to different sets of environments.
Move projects into groups with projects that deploy to shared environments.

## Too many steps

> A deployment process should have 20 or fewer steps.

Complex deployments can often involve many steps. However, deployment processes can become brittle and hard to manage when they include too many steps.

You can create runbooks to encapsulate steps into a reusable unit or use separate projects to reduce the number of steps in a deployment process.

## Direct tenant references

> Explicit groups should be created to manage tenants.

Tenants can often be grouped based on shared properties like regions, release rings, performance tiers, etc. These groupings are best expressed as tenant tags.

However, it is also possible to refer to groups of tenants directly, creating an implicit group of tenants. This can be seen when the same group of tenants is directly referenced across multiple resources like accounts, certificates, and targets. Replacing these implicit groups of tenants with explicit tenant tags improves the manageability of the Octopus instance.

Use tags to group tenants and use these tags rather than specific tenants for accounts, certificates, and targets.

## Unhealthy targets

> Targets should not remain unhealthy for long periods.

Targets include health checks that indicate if they are online and healthy. Unhealthy targets are still included in deployments by default but are unlikely to complete the deployment successfully. Targets that have been unhealthy for some time likely need attention.

Resolve health issues with targets, or remove them if they are no longer used.

## Shared Git usernames

> Projects should use unique Git usernames.

Config-as-code-enabled projects can either define project-specific Git credentials or reference shared credentials. Projects that define the same Git username likely indicate that they are duplicating Git credentials, which makes the Octopus instance harder to maintain.

Create shared Git credentials to use in place of project-specific Git credentials.

## Deployment queue times

> Deployments should queue for less than a minute.

The Octopus task cap limits how many tasks are executed in parallel. When the task queue exceeds the task cap, tasks wait for their turn to be started. If too many tasks take too long to start, deployments and runbooks can be delayed.

Increase the task cap, or if running Octopus HA, add another Server node to spread tasks between instances.

## Unused targets

> Targets should be actively used.

Targets are used to represent where deployments are performed. Targets count towards license limits and often have health check tasks scheduled at regular intervals.

Targets not used in the last month may no longer be required while still creating tasks and contributing to the active target count on your license.

Remove any targets that are no longer used.

## Lifecycle retention

> Retention policies should be used to clean up old resources.

Lifecycle retention rules provide the option to specify that resources like releases and packages are retained for a specific time or retain the last specified number of resources. Lifecycles can also be configured to retain resources indefinitely.

Retaining resources indefinitely is inefficient and can introduce performance issues over time.
Use retention rules to clean up old resources to improve performance.

## Good practices

You don't have to remember all these good practices, use Octolint to do the hard work, so you have more time available to implement improvements.

Whether you improve your security by replacing API keys that don't expire or using step templates to convert complex processes into reusable templates, Octolint is a good way to find out what you can improve and make sure things stay in good shape.

Happy deployments!
