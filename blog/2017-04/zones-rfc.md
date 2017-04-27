---
title: "Zones RFC"
description: We are designing a new feature to allow promoting Releases between Octopus Servers (e.g. for security or geographic reasons). This is a request-for-comments.  
author: michael.richardson@octopus.com
visibility: private
tags:
 - RFC 
---

# The problem (Michael)

- Security/PCI
- Geographic Regions

# Proposed solution

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

# Super nitty gritty

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

# Superseded Solutions (Vanessa)

- Octopus Migrator
- Offline Drops
- Octo.exe export/import
- Custom scripting using the Octopus REST API
- Manually migrating everything

# Rollout

- Phases
- Octopus v4?

# Feedback
