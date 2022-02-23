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

Not everything is moved into the repository when you enable version control. A list of version-controlled resources is available in our [configuration as code reference](https://octopus.com/docs/projects/version-control/config-as-code-reference).

:::warning
Switching on version control for your project is a one-way change. You can't move the project back into the database once it's in a repository. You can clone an existing project to try config as code and confirm that it meets your needs before enabling it for your production projects.
:::

## Where to store your configuration

One of the first decisions you will need to make is where to store your config as code. You can choose between several options. You can keep your configuration:

- Alongside your application code
- In a separate application-specific deployment repository
- In a repository that contains all the configurations for an Octopus Space
- In a central deployment configuration repository

Each option is described below to explain when they work and when to avoid them. You might have noticed that you can arrange these possibilities along a scale from a one-to-one relationship with applications to a single large repository. We recommend keeping your deployment configuration in the same repository as the application code, but there are specific circumstances where the other options may be suitable.

### Alongside application code

Placing your deployment configuration alongside your application code is the pattern we recommend. It is best  _to evolve your deployment process alongside your application code_. Putting the configuration in the same location as the application aligns with DevOps practices, where engineers take end-to-end responsibility for their applications.

If you opt to store your configuration in the application repository, each application will have its own `.octopus` directory with the configuration files.

This approach benefits from keeping the configuration in the same location as the application code, so you'll be able to find it quickly and easily. It will have the same process and security policy as code changes for the application.

You may wish to avoid this pattern if you have a large application codebase, which may affect the performance when you switch between branches in the Octopus app. Currently, we need to retrieve the new branch to obtain the configuration, but we are actively working on improving the performance in this area.

You will also need to mask the `.octopus` folder to avoid triggering a build for each configuration change.

This pattern suits an autonomous team responsible for both the application and its deployment.

### Application-specific deployment repository

This option adds an additional repository per application to store the application's deployment configuration. For example, the application "OctoPetShop" would have an extra repository named "OctoPetShop Deployments" that contains only the config as code files. Each deployment repository would have a `.octopus` directory containing the OCL files. You can use a naming convention to make it easier to find the deployment configuration related to a specific application.

Using this approach, you can easily grant separate permissions to the deployment configuration and apply different policies for pull requests. Each branch will remain as small as possible, and it will be fast when you switch between branches in the Octopus app.

This approach will mean you double the number of repositories under management, making it undesirable if you have many applications.

### Deployment repository grouped by space

It is possible to store many deployment projects within a single repository using sub-folders. Instead of storing the OCL files directly within the `.octopus` directory, each can have a folder such as `.octopus/OctoPetShop/` or `.octopus/OctoHR`. You can create a single repository to store all of the configurations for an Octopus Space.

This approach benefits from the existing design work you have already undertaken to arrange your spaces. It reduces the total number of repositories you need to manage while still allowing you to set specific permissions to the deployment repository.

### Central deployment repository

Taking the grouped approach a step further, you _could_ have a single repository to store all of your deployment configurations. You will have fewer repositories but may suffer from issues as the number of branches increases. If you had ten projects within the repository and each had five branches, you would see 50 items in the branch selector in the Octopus app.

Unless you have a small number of applications, you should avoid this option. In general, if you have started using spaces, you should consider organizing your repositories to match them.

## Using config as code effectively

Use branches to contain your risk.

Make edits in the Octopus app and commit the changes with a comment.

Review the changes using your pull request process and fix any typos in a text editor.

CAN YOU HAVE A PROJECT THAT ONLY DEPLOYS TO DEV WORKING FROM A BRANCH SEPARATE TO THE DEFAULT MAIN BRANCH USED FOR THE FULL PIPELINE?

## When config as code is the wrong option

Many deployment resources don't belong to a project, for example; spaces, tenants, environments, and accounts. We didn't intend for config as code to handle the version control of these items.

You can use the [Octopus Deploy Terraform Provider](https://registry.terraform.io/providers/OctopusDeployLabs/octopusdeploy/latest/docs) to handle these resources. You can find out [how to get started with the Terraform provider for Octopus Deploy on our blog](https://octopus.com/blog/octopusdeploy-terraform-provider).

You might also consider sharing a single configuration between multiple projects to keep the process in sync. However, this requires many non-project resources to be kept identical between projects, which quickly becomes hard to manage.

Instead of sharing the same OCL files between multiple projects, you should create a custom tool to interact with the [Octopus Deploy REST API](https://octopus.com/docs/octopus-rest-api)to enforce the desired process configuration. You can read more in our documentation for [synchronizing multiple instances](https://octopus.com/docs/administration/sync-instances).

## Conclusion

While there is no single correct way to use config as code, some suitable heuristics allow you to choose between four common Git repository strategies. We also reviewed cases where config as code isn't the intended solution. Instead, we provided alternative mechanisms to sync resources outside the project scope and keep deployment processes in sync.