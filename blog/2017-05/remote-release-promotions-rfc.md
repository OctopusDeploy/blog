---
title: "Remote Release Promotions RFC"
description: We are designing a new feature to allow promoting Releases between different Octopus Servers (Spaces). You may want to do this for for security, geographic, or other reasons. This is a request-for-comments.  
author: michael.richardson@octopus.com
visibility: private
tags:
 - RFC 
---

## The problem

There are scenarios where it makes sense for different Octopus Server instances to perform deployments depending on which environment is being deployed to.

For example:

- Octopus Server 1 deploys to Development and Testing environments
- Octopus Server 2 deploys to Staging and Production environments

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

Imagine if you could add a space to your lifecycle and then promote a release to another space, just like you can to an environment. When you promote a release to another space, Octopus could bundle up everything required to deploy that release into the environments in the **other** space. That's why we're calling this feature **Remote Release Promotions**.

- High-level architecture diagram (pretty picture) (Vanessa)
- Release lifecycle (showing promotion through environments, then zones) (Vanessa)

## Definitions

In the rest of this RFC we are going to introduce some new terms so we don't all get horribly confused.

- Space: Contains a set of Projects, Environments, Variables, Deployment Targets, etc, bounded by a single Octopus database. Learn more in our recent [RFC](/blog/2017-05/odcm-rfc.md).
- Release Bundle: A package containing everything required to deploy a specific Release of a Project.
- Deployment Receipt: A document containing everything required to show the result of deploying a specific Release of a Project.
- Source Space: The Space that owns the Project and its Releases, and where Release Bundles are created if you decide to cross Space boundaries.
- Target Space: The Space where a Release Bundle will be imported. The Release extracted from the Release Bundle can then be deployed to Environments in this Space.
- Remote Environment: A reference to an Environment owned by another Space.
- Remote Project: A reference to a Project owned by another Space.
- Variable Template: We introduced this concept with multi-tenant deployments. In this context you could express that a variable value is required for each Environment a Project can be deployed into.

## Example: Secure Environments

Let's explore this concept using the Secure Environments example we mentioned earlier, where you want strict separation between your development and production environments. In this case we will model this separation using two Spaces:

- `DevTest Space`: where your application is deployed for development and testing purposes
- `Prod Space`: where the production deployments of your application will be deployed and strict compliance controls are required

_IMAGE: Show two spaces, indicating where project, and each environment is owned, and how the bundle flows_

Let's consider how each different person in your organization might interact with Octopus to promote a release across these two Spaces all the way to production.

### Persona: Project contributor

Project contributors are people who configure the deployment process and variables of a project. Typically these are your software developers who are also writing the source code and building your packages. In this case we don't see very much changing - life will pretty much go on just like before.

However, the `Production` environment is owned by the `Prod Space`, meaning the `DevTest Space` has no concept of this environment:

- How do you provide variable values that will be used when deploying to the `Production` environment?
- How do you configure special steps of your deployment process so they only execute when deploying to the `Production` environment?

Please welcome variable templates and remote environments!

#### Variable templates

Imagine if you are the person importing a Release Bundle - how do you know which variables need values? And even if you know which variables you need to set, what should you set the value to?

Now imagine as a project contributor, you could express that a variable value is required for each environment a project can be deployed into. And imagine you could define a data type for the variable, provide help text, decide whether the value is mandatory or optional, or even provide a default value.

Variable templates could make it much easier for a person importing a Release Bundle into their space to "fill in the blanks".

:::hint
This would also be really handy even if you are only promoting releases within your own space. Using variable templates, if you introduce a new environment into your own space, Octopus will prompt you for those variable values.
:::

We introduced the concept of [variable templates](https://octopus.com/docs/deploying-applications/variables/variable-templates) for multi-tenant deployments in Octopus 3.4. We would like to build on this concept further as part of this set of features.

_IMAGE: Variable Template Editor?_

Learn more: _LINK: Variable Template GitHub Issue_

#### Remote Environments

**Steps for Remote Environments**: In some cases you want certain steps to be executed in the `Production` environment. But now that the `Production` environment is owned by the `Prod Space`, your `DevTest Space` doesn't know the `Production` environment exists! How can you tailor your deployment process for environments owned by other spaces? Imagine if you could add a **Remote Environment** to the `DevTest Space`. This remote environment would be like a placeholder for the real `Production` environment, and Octopus could name it `Prod Space: Production` so we are all clear about the ownership of this environment. _Think of this like namespaces: so you can have a `Production` environment in multiple spaces._ Now you would be able to scope steps to `Prod Space: Production`, and those steps will be run when a release is eventually deployed to that environment.

**Variable values for Remote Environments**: We can also imagine a case where you already know a handful of the variable values required for the `Production` environment (perhaps they aren't secret). Now you would be able to set those values in your `DevTest Space`, scope them to `Prod Space: Production` and they will be used when a release is eventually deployed to that environment.

### Persona: Release bundler

- Bundles the release to be promoted to a specific remote space
- Could be the same person as Project Contributor, or could be the same person as Release Acceptor/Approver/Deployer depending on your security model

### Persona: Release acceptor

- Imports the release bundle
- Adds the missing variable values
- Has permissions to create projects, edit variables, add packages, etc

### Persona: Release approver(s)

- Idea for multi-team sign off on a release before it is allowed to be deployed

### Persona: Release deployer

- Actually deploys the release to the environment(s)

Now the release has been accepted it can be deployed to the environments in the `Prod Space`. For all intents and purposes this would work just like the release was created in the `Prod Space`: all the same rules would apply for promoting this release including:

- Project permissions: teams could be restricted to **Remote Projects** just like normal projects - after all, they are just normal projects but owned by another space, and namespaced just like **Remote Environments**
- Environment permissions: teams could be granted permissions to environments in the `Prod Space` just like normal
- Lifecycle progression: the release should be deployed to a `Staging` environment before being deployed to the `Production` environment

#### Lifecycles

We talked earlier about lifecycles, but it's worth mentioning again: Lifecycles will be self-contained within a Space. This gives the teams in each space the ability to manage their own environments and lifecycles how they see fit. For example, you might decide to introduce a `Staging` environment into the `Prod Space` as a pre-requisite to deploy a release into `Production` - just like a normal lifecycle. And it would be nice if the decision to introduce `Staging` had minimal impact (if any) on the `DevTest Space`.


## Persona: Project manager

- Wants to see an aggregated overview of the deployments for the entire lifecycle

## Persona: Operations

- Has to set up the connection/trust/relationship between the two spaces, potentially over an air gap

## Release bundle

- What goes across?
- Packages (hash)

## Release Acceptance

- Review and accept diffs?
- Provide variable values

## Flowing deployment results back across

- Deployment receipts
- Updating the source dashboard

## Variables

- Environment Variable Templates

## Super nitty gritty

- Disconnected mode (we won’t force you to use connected)
- Version tolerance/message schema
- Snapshots (how to update on the source zone and re-promote to remote zone)
- What will be locked on the target zone?
- Matching on names, not IDs
- We want to use delta compression
- Tenants could span zones
- Lifecycles won’t span zones

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
