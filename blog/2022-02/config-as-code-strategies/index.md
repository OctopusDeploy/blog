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

Since last year, when we released the preview of Config as Code...





﻿When we talk to customers about config as code they often ask what the guidance is for certain things.

I﻿s the intention of the setup 1:1 ratio project to repo?﻿
What's the recommendation for branching use?
﻿It would be good to have an opinionated article around this.

The existing material suggests the intended approach is to have a "configuration repository" is this correct and what are the reasons? Is there a case where keeping the config alongside the application code is appropriate?

Reasons for a configuration repository:
The benefit of independent configuration changes.
A tailored deployment process.
Ease of change audits.
Additional security controls, which are usually used for application secrets that a developer may not need access to.
The configuration is versioned.
It’s accessible to only those who need it.
Secrets are encrypted.
Proper approval gates are in place.

Note: Octopus Deploy extension for VSCode
https://marketplace.visualstudio.com/items?itemName=octopusdeploy.vscode-octopusdeploy

## Why config as code

History: Git is a time-machine for code, and being able to view the what, when, and who for your Octopus configuration alongside application code is undeniably useful.

visibility/traceability in CI/CD pipeline configuration

Branching: Today, there is a single instance of the deployment process. This makes testing changes difficult, as when a release is created it will use the current deployment process. Git branches make it possible to have as many versions of the deployment process as you like, allowing iterating on changes without impacting stability.

Single source of truth: Having application code, build scripts, and deployment configuration all living together makes everyone feel warm and fuzzy.

Cloning: Imagine being able to copy a .octopus folder to a brand new Git repository (maybe change a few variables) and use that as the Octopus project template.

Key point - it doesn't stop you using the Octopus UI - you can make changes in the UI and commit them to the repository.

## Folder per project

Folder per project is a requirement, but you don't have to put them in a different repo - you could have one repo with many project folders.

What are the good practices here - one repo per space?

## Use branches and pull requests

Use the features to review and accept changes.

Use branches to test changes in a pre-live environment before merging the changes into your main line.

## Where to put the config

### Alongside application code

Deployment configuration in the same repository as the application. Is this is the intended design...for what reasons?

Benefits
 - All related information is in the same place
 - Your existing process for code reviews and security can include the deployment process

Considerations
- When you change the deployment config it will trigger a build as usual if the folder is included in the build triggers
- If there is a lot of application code in the repository it might impact performance as Octopus needs to get the branch, switching branches could take longer


### In a separate deployment repository

Central deployment configuration repository.

This would suit situations where the same team manages all the deployments, rather than the app teams.

You have to be careful as if there are 10 projects and four branches get created to handle changes being made to each project, you now have a list of 40 branches to choose from in the Octopus Deploy UI.

### Can you change your mind and move it?

You can transfer the configuration to a new folder, or to a new Git repository.

If you decide to move your configuration, the history won't be moved with it, only the current deployment process state.

Q: what warnings or caveats should we add - making sure any branch names you depend on are maintained.

#### Moving config files into a folder

You will need to pause changes to the deployment configuration while you perform this operation.

Ensure you have the latest version of the config files.
Create the new folder under the .octopus directory.
Copy the files into the new folder

Open the project in Octopus Deploy
Open Settings / Version Control
Expand the Git FIle Storage Directory and update the folder name
Click SAVE to store the changes
Check your deployment process
Delete the files from the old location


#### Moving config files to a new repository

Create a new Git repository
Copy the latest version of the .octopus folder into the new repository
Commit and push the changes

Open the project in Octopus Deploy
Open Settings / Version Control
Update the Git Repository address
Enter a personal access token for the new Git repository
Click TEST to check your connection to the repository
Click SAVE to store the changes
Check your deployment process
Delete the files from the old location

## When to use the Terraform provider instead

For things outside of projects, like spaces, tenants, environments, accounts, etc (this came up multiple times on the community slack)