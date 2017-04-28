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
- Security
- Geography

### Security
For security purposes many organizations separate their production and development environments. A common driver for this is achieving PCI DSS compliance. 

The secure zone may even be completely disconncted (aka air-gap).

These organisations still want all of the Octopus-goodness, like promoting the same release through the environments, and seeing the progression on the dashboard.  But they don't want the development Octopus server to be able to connect to the production environment.  They also likely want a different set of users (possibly from a distinct Active Directory domain) to be have permissions to the production Octopus instance.

*TODO: INSERT PRETTY PICTURE*

### Geography
Other organisations may deploy to geographically-distant environments. 

For example, their development environment may be located in Brisbane, while their production environments live in data centers in the US and Europe. 

The problem with this currently, is that the packages are transferred at deployment time.  These customers would like to be able to promote the release at a time of their choosing (including transferring the packages), then perform the deployment with the packages already located in the appropriate Octopus instance, close to the target machines.

*TODO: INSERT PRETTY PICTURE*



## Proposed solution

 - High-level architecture diagram (pretty picture) (Vanessa)
 - Release lifecycle (showing promotion through environments, then zones) (Vanessa)
 - A day-in-the-life of a release (so people can identify with it) (Mike)
   - Nothing much changes for project contributors

## Zones (introduce the concept)

- Promoting a release to a zone (separate lifecycles)?

## Remote Environments

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
