---
title: "Zones RFC"
description: We are designing a new feature to allow promoting Releases between Octopus Servers (e.g. for security or geographic reasons). This is a request-for-comments.  
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

- Secure Environments
- Geographically distant environments
- Teams who share services

### Secure environments

For security purposes many organizations separate their production and development environments. A common driver for this is achieving PCI DSS compliance.

The secure zone may even be completely disconnected (aka air-gap).

These organizations still want all of the Octopus-goodness, like promoting the same release through the environments, and seeing the progression on the dashboard.  But they don't want the development Octopus server to be able to connect to the production environment.  They also likely want a different set of users (possibly from a distinct Active Directory domain) to be have permissions to the production Octopus instance.

*TODO: INSERT PRETTY PICTURE*

### Geographically distant environments

Other organizations may deploy to geographically-distant environments.

For example, their development environment may be located in Brisbane, while their production environments live in data centers in the US and Europe.

The problem with this currently, is that the packages are transferred at deployment time.  These customers would like to be able to promote the release at a time of their choosing (including transferring the packages), then perform the deployment with the packages already located in the appropriate Octopus instance, close to the target machines.

*TODO: INSERT PRETTY PICTURE*

### Teams who share services

TODO: Like SOA or Microservice teams. "I want to deploy an instance of your services in my environment so I can test my service against yours."

*TODO: INSERT PRETTY PICTURE*

## Proposed solution

- High-level architecture diagram (pretty picture) (Vanessa)
- Release lifecycle (showing promotion through environments, then zones) (Vanessa)
- A day-in-the-life of a release (so people can identify with it) (Mike)
  - Nothing much changes for project contributors

Our proposed solution leverages the idea of Spaces, where each Space has its own set of Users, Projects, Environments, Lifecycles, etc. Now imagine if you could add Spaces to your Lifecycles. When you promote a release to another Space, Octopus could bundle up everything required to deploy that Release of your Project to the Environments in the other Space.

## Definitions

In the rest of this RFC we are going to introduce some new terms so we don't all get horribly confused.

- Space: Learn more in the [RFC](/blog/2017-05/odcm-rfc.md)
- Release Bundle: A package containing everything required to deploy a specific Release of a Project.
- Deployment Receipt: A package containing everything required to show the result of deploying a specific Release of a Project.
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

### Project contributor

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

#### Remote Environments

In some cases you want certain steps to be executed in the `Production` environment. But now that the `Production` environment is owned by the `Prod Space`, your `DevTest` space doesn't know the `Production` environment exists! How can you tailor your deployment process for environments owned by other spaces?

Imagine if you could add a **Remote Environment** to your space, as a placeholder for the real `Production` environment. Now you would be able to and scope steps to that remote environment, and those steps will be run when a release is eventually deployed to that remote environment.

We can also imagine a case where you already know a handful of the variable values required for the `Production` environment (perhaps they aren't secret) - now you would be able to set those values in your `DevTest Space`, and they will be used when deploying to the `Production` environment in the `Prod Space`.

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
