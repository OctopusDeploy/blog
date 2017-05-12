---
title: "Remote Release Promotions RFC"
description: We are designing a new feature to allow promoting Releases between different Octopus Servers (Spaces). You may want to do this for for security, geographic, or other reasons. This is a request-for-comments.  
author: michael.richardson@octopus.com
visibility: private
tags:
 - RFC 
---

## The problem

There are scenarios where it makes sense for different Octopus Server instances to perform deployments depending on which environment is being deployed to. For example:

- Octopus Server 1 deploys to Development and Testing environments
- Octopus Server 2 deploys to Staging and Production environments

### Elevator pitch

We are planning a feature which enables you to promote releases across multiple Octopus Servers... in a nice way. :) If you are trying to do this today, you know it hurts real good.

*TODO: INSERT PRETTY PICTURE HERE*

The two most common reasons for this are:

- Secure environments
- Geographically distant environments

### Secure environments

For security purposes many organizations separate their production and development environments. A common driver for this is achieving PCI DSS compliance.

The secure zone may even be completely disconnected (aka air-gap).

These organizations still want all of the Octopus-goodness, like promoting the same release through the environments, and seeing the progression on the dashboard.  But they don't want the development Octopus server to be able to connect to the production environment.  They also likely want a different set of users (possibly from a distinct Active Directory domain) to be have permissions to the production Octopus instance.

*TODO: INSERT PRETTY PICTURE*

### Geographically distant environments

Other organizations may deploy to geographically-distant environments.

For example, their development environment may be located in Brisbane, while their production environments live in data centers in the US and Europe.

The problem with this currently, is that the packages are transferred at deployment time.  These customers would like to be able to promote the release at a time of their choosing (including transferring the packages), then perform the deployment with the packages already located in the appropriate Octopus instance, close to the target machines.

## Proposed solution

Our proposed solution will enable you to spread your entire deployment lifecycle across multiple "Spaces". A "Space" is a concept we are [planning to introduce](/blog/2017-05/odcm-rfc.md), where each "Space" has its own set of projects, environments, lifecycles, teams, permissions, etc.

Imagine if you could add a Space to your Lifecycle and then promote a release to another Space. When you promote a release to another Space, Octopus could bundle up everything required to deploy that release into the environments in the **other** Space. We will also cater for scenarios where there is strict separation between your Spaces (think PCI DSS). That's why we're calling this feature **Remote Release Promotions**.

- High-level architecture diagram (pretty picture) (Vanessa)
- Release Lifecycle (showing promotion through environments, then zones) (Vanessa)

### Lifecycles

We think Lifecycles should be self-contained within a Space. This gives the teams in each Space the ability to manage their own environments and Lifecycles how they see fit. For example, a member of one Space might decide to introduce an environment into their Lifecycle. We don't want the decision to introduce an environment into a Lifecycle in one Space to have any impact on any other Spaces.

We also think Lifecycles should be able to model interesting progressions that cross Space boundaries. Think of it like building your overall deployment lifecycle by composing multiple Octopus Lifecycles together:

1. You might want to promote a release through your test environments, then promote the release to one or more Spaces that manage the production environments.
1. You might want to promote a release through your dev team's test environments, then promote the release to another Space managed by a QA team. When they are finished testing you want the dev team to promote that same release to yet another Space that manages your production environments.
1. You might want to do the same as #2, but once the QA team is finished they are responsible to promote the release to the production Space without going back through the dev team.

## Definitions

In the rest of this RFC we are going to introduce some new terms so we don't all get horribly confused.

- Space: Contains a set of projects, environments, variables, teams, permissions, etc, bounded by a single Octopus database. Learn more in our recent [RFC](/blog/2017-05/odcm-rfc.md).
- Release Bundle: A package containing everything required to deploy a specific release of a project.
- Deployment Receipt: A document containing everything required to show the result of deploying a specific release of a project.
- Source Space: The Space that owns the project and its releases, and where release bundles are created if you decide to cross Space boundaries.
- Target Space: The Space where a release bundle will be imported. The release extracted from the release bundle can then be deployed to environments in this Space.
- Remote Environment: A reference to an environment owned by another Space.
- Remote Project: A reference to a project owned by another Space.
- Variable Template: We introduced this concept with multi-tenant deployments. In this context you could express that a variable value is required for each environment a project can be deployed into.

### Trusting other Spaces

- Has to set up the connection/trust/relationship between the two Spaces, potentially over an air gap

## Example: Secure Environments

Let's explore this concept using the **Secure Environments** example we mentioned earlier, where you want strict separation between your development and production environments. In this case we will model this separation using two Spaces:

- `DevTest Space`: where your application is deployed for development and testing purposes
- `Prod Space`: where the production deployments of your application will be deployed and strict compliance controls are required

_IMAGE: Show two Spaces, indicating where project, and each environment is owned, and how the bundle flows_

Let's consider how each different person in your organization might interact with Octopus to promote a release across these two Spaces all the way to production.

### Working with projects

We don't see very much changing - life will pretty much go on just like before. You will still be able to change the deployment process, manage variables, and create and deploy releases to environments in the `DevTest Space` just like normal. However in this example the `Production` environment is owned by the `Prod Space`, meaning the `DevTest Space` has no concept of this environment:

- How do you provide variable values that will be used when deploying to the `Production` environment?
- How do you configure special steps of your deployment process so they only execute when deploying to the `Production` environment?

Please welcome variable templates and remote environments!

#### Variable templates

Imagine if you are the person importing a release bundle into your Space - how do you know which variables need values? And even if you know which variables you need to set, what should you set the value to?

Now imagine as a project contributor, you could express that a variable value is required for each environment a project can be deployed into. And imagine you could define a data type for the variable, provide help text, decide whether the value is mandatory or optional, or even provide a default value.

Variable templates could make it much easier for a person importing a release bundle into their Space to "fill in the blanks".

:::hint
This would also be really handy even if you are only promoting releases within your own Space. Using variable templates, if you introduce a new environment into your own Space, Octopus will prompt you for those variable values.
:::

We introduced the concept of [variable templates](https://octopus.com/docs/deploying-applications/variables/variable-templates) for multi-tenant deployments in Octopus 3.4. We would like to build on this concept further as part of this set of features.

_IMAGE: Variable Template Editor?_

Learn more: _LINK: Variable Template GitHub Issue_

#### Remote Environments

**Steps for Remote Environments**: In some cases you want certain steps to be executed in the `Production` environment. But now that the `Production` environment is owned by the `Prod Space`, your `DevTest Space` doesn't know the `Production` environment exists! How can you tailor your deployment process for environments owned by other Spaces? Imagine if you could add a **Remote Environment** to the `DevTest Space`. This remote environment would be like a placeholder for the real `Production` environment. Octopus could even name it `Prod Space: Production` so we are all clear about the ownership of this environment. _Think of this like namespaces: so you can have a `Production` environment in multiple Spaces._ Now you would be able to scope steps to `Prod Space: Production`, and those steps will be run when a release is eventually deployed to that environment.

**Variable values for Remote Environments**: We can also imagine a case where you already know a handful of the variable values required for the `Production` environment (perhaps they aren't secret). Now you would be able to set those values in your `DevTest Space`, scope them to `Prod Space: Production` and they will be used when a release is eventually deployed to that environment.

### Publishing releases to other Spaces (Mike N)

- Bundles the release to be promoted to a specific remote Space
- Could be the same person as Project Contributor, or could be the same person as Release Acceptor/Approver/Deployer depending on your security model

### Subscribing to releases from other Spaces (Mike N)

### Importing releases into your Space (Mike N)

- Import the release bundle
- Choose a lifecycle
- Add the missing variable values
- Person must have permissions to create projects, edit variables, add packages, etc

### Approving a release (Mike N)

- Idea for multi-team sign off on a release before it is allowed to be deployed - no need for manual intervention steps for this purpose

### Deploying releases (Mike N)

Now the release has been accepted it can be deployed to the environments in the `Prod Space`. For all intents and purposes this would work just like the release was created in the `Prod Space`: all the same rules would apply for deploying this release including:

- Project permissions: teams could be restricted to **Remote Projects** just like normal projects - after all, they are just normal projects but owned by another Space, and namespaced in the same way **Remote Environments** will be namespaced
- Environment permissions: teams in the `Prod Space` could be granted appropriate permissions to environments in the `Prod Space`, just like normal
- Lifecycle progression: Octopus will ensure each release progresses through the appropriate Lifecycle in the `Prod Space`, just like normal

### Aggregated dashboard (Mike N)

- Wants to see an aggregated overview of the deployments for the entire lifecycle

## Release bundle (Michael R)

- What goes across?
- Packages (hash)

## Release Acceptance (Michael R)

- Review and accept diffs?
- Provide variable values

## Flowing deployment results back across (Michael R)

- Deployment receipts
- Updating the source dashboard

## Variables (Michael R)

- Environment Variable Templates

## Super nitty gritty (Michael R)

- Disconnected mode - details on how we see this working.
- Version tolerance/message schema
- Snapshots (how to update on the source Space and re-promote to remote Space)
- What will be locked on the target Space?
- We want to use delta compression
- Tenants could span Spaces
- Channels in remote Spaces
- ARC in remote Spaces

## Security Concerns

- Two-way trust using PKI (like Octopus and Tentacle)

## Superseded Solutions (Vanessa)

- Octopus Migrator
- Offline Drops
- Octo.exe export/import
- Custom scripting using the Octopus REST API
- Manually migrating everything

## Rollout

- Phases
- Octopus v4?

## Feedback
