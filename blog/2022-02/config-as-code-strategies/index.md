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

Since last year, when we released the early-access preview of Octopus Config as Code, there have been many questions about how to use the feature to get the best results. This article will explain some good practices for using config as code and how to adjust your strategy in different situations.

## Why use config as code?

Git is the perfect solution for versioning code and keeping track of changes over time. It has established patterns for branching the code and publishing and approving changes. It also allows you to compare versions and travel back in time if you need to.
Using Git to version control your config as code, you can:

- Branch your configuration and test changes in the branch before merging them
- Review and collaborate on changes using pull requests
 - Clone an existing project to use as a template for future projects
 - Track changes to the deployment configuration using the same tools you already use for your application code
- Edit your deployment configuration in your preferred text editor or within the Octopus app

The configuration is stored as human-readable Octopus Configuration Language (OCL) files to make it easier to read and edit the deployment process and review any changes. There is a [Visual Studio Code extension to make it easier to work with OCL files](https://marketplace.visualstudio.com/items?itemName=octopusdeploy.vscode-octopusdeploy).

## Where to store your configuration

One of the first decisions you will need to make is where to store your config as code. You can choose between several options. You can keep your configuration:

- Alongside your application code
- In a separate application-specific deployment repository
- In a repository that contains all the configurations for an Octopus Space
- In a central deployment configuration repository

Each option is described below to explain when they work and when to avoid them. You might have noticed that you can arrange these possibilities along a scale from a one-to-one relationship with applications to a single large repository. There is a natural trade-off between having many repositories versus having potentially slower performance in fewer large repositories.

### Alongside application code

If you opt to store your configuration in the application repository, each application will have its own `.octopus` directory with the configuration files.

This approach benefits from keeping the configuration in the same location as the application code, so you'll be able to find it quickly and easily. It will have the same process and security policy as code changes for the application.

You may wish to avoid this pattern if you have a large application code base as this may affect the performance when you switch between branches in the Octopus app, as it needs to retrieve the new branch to obtain the configuration. You will also need to mask the `.octopus` folder to avoid triggering a build for each configuration change.

This pattern suits an autonomous team responsible for both the application and its deployment.

### Application-specific deployment repository

This option adds an additional repository per application to store the application's deployment configuration. For example, the application "OctoPetShop" would have an extra repository named "OctoPetShop Deployments" that contains only the config as code files. Each deployment repository would have a `.octopus` directory containing the OCL files. You can use a naming convention to make it easier to find the deployment configuration related to a specific application.

Using this approach, you can easily grant separate permissions to the deployment configuration and apply different policies for pull requests. Each branch will remain as small as possible, and it will be fast when you switch between branches in the Octopus app.

This approach will mean you double the number of repositories under management, making it undesirable if you have many applications.

### Deployment repository grouped by space

It is possible to store many deployment projects within a single repository using sub-folders. Instead of the OCL files being stored directly within the `.octopus` directory, each can have a folder such as `.octopus/OctoPetShop/` or `.octopus/OctoHR`. You can create a single repository to store all of the configurations for an Octopus Space.

This approach benefits from the existing design work you have already undertaken to arrange your spaces. It reduces the total number of repositories you need to manage while still allowing you to set specific permissions to the deployment repository.

### Central deployment repository

Taking the grouped approach a step further, you _could_ have a single repository to store all of your deployment configurations. You will have fewer repositories but may suffer from issues as the number of branches increases. If you had ten projects within the repository and each had five branches, you would see 50 items in the branch selector in the Octopus app.

Unless you have a small number of applications, you should avoid this option. In general, if you have started using spaces, you should consider organizing your repositories to match them.

## When config as code is the wrong option

Many deployment resources don't belong to a project, for example; spaces, tenants, environments, and accounts. We didn't intend for config as code to handle the version control of these items.

You can use the [Octopus Deploy Terraform Provider](https://registry.terraform.io/providers/OctopusDeployLabs/octopusdeploy/latest/docs) to handle these resources. You can find out [how to get started with the Terraform provider for Octopus Deploy on our blog](https://octopus.com/blog/octopusdeploy-terraform-provider).

You might also consider sharing a single configuration between multiple projects to keep the process in sync. However, this requires many non-project resources to be kept identical between projects, which quickly becomes hard to manage.

Instead of sharing the same OCL files between multiple projects, you should create a custom tool to interact with the [Octopus Deploy REST API ](https://octopus.com/docs/octopus-rest-api)to enforce the desired process configuration. You can read more in our documentation for [synchronizing multiple instances](https://octopus.com/docs/administration/sync-instances).

## Conclusion

While there is no single correct way to use config as code, some suitable heuristics allow you to choose between four common Git repository strategies. We also reviewed cases where config as code isn't the intended solution. Instead, we provided alternative mechanisms to sync resources outside the project scope and keep deployment processes in sync.


## POSSIBLY A SEPARATE POST?


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
