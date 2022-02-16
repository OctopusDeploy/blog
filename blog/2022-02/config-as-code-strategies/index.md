---
title: Config as Code strategies
description: TBC
author: steve.fenton@octopus.com
visibility: private
published: 9999-01-01
metaImage: 
bannerImage: 
bannerImageAlt: TBC
isFeatured: false
tags:
 - DevOps
 - Configuration as Code
---

The existing material suggests the intended approach is to have a "configuration repository" is this correct and what are the reasons? Is there a case where keeping the config alongside the application code is appropriate?

Reasons for a configuration repository:
The benefit of independent configuration changes.
A tailored deployment process.
Ease of change audits.
Additional security controls, which are usually used for application secrets that a developer may not need access to.

Note: Octopus Deploy extension for VSCode
https://marketplace.visualstudio.com/items?itemName=octopusdeploy.vscode-octopusdeploy

## Why config as code

History: Git is a time-machine for code, and being able to view the what, when, and who for your Octopus configuration alongside application code is undeniably useful.

Branching: Today, there is a single instance of the deployment process. This makes testing changes difficult, as when a release is created it will use the current deployment process. Git branches make it possible to have as many versions of the deployment process as you like, allowing iterating on changes without impacting stability.

Single source of truth: Having application code, build scripts, and deployment configuration all living together makes everyone feel warm and fuzzy.

Cloning: Imagine being able to copy a .octopus folder to a brand new Git repository (maybe change a few variables) and use that as the Octopus project template.


## Alongside application code

Deployment configuration in the same repository as the application. Is this is the intended design...for what reasons?

Benefits
 - All related information is in the same place
 - Your existing process for code reviews and security can include the deployment process

Considerations
- When you change the deployment config it will trigger a build as usual if the folder is included in the build triggers


## Deployment repository

Central deployment configuration repository.

This would suit situations where the same team manages all the deployments, rather than the app teams.