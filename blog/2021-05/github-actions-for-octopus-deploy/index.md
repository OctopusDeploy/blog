---
title: Announcing GitHub Actions for Octopus Deploy
description: GitHub Actions for Octopus Deploy are now available enabling you to integrate your GitHub builds with Octopus deploys.
author: rob.pearson@octopus.com
visibility: public
published: 2022-06-01
metaImage: github-actions-integration
bannerImage: github-actions-integration
tags:
 - DevOps
 - GitHub Actions
---

We're proud to announce we've shipped our first official GitHub Actions for Octopus Deploy. This is an initial batch of actions and they cover the core integration scenarios so you can integrate your GitHub builds with your Octopus deploys and runbook runs. We plan to add additional GitHub Actions later in the year. 

- [Push packages](https://github.com/marketplace/actions/push-package-to-octopus-deploy) to Octopus Deploy. 
- [Create releases](https://github.com/marketplace/actions/create-release-in-octopus-deploy) in Octopus Deploy.
- [Execute a runbook](https://github.com/marketplace/actions/run-runbook-in-octopus-deploy) in Octopus Deploy.
- [Install](https://github.com/marketplace/actions/install-octopus-cli) the Octopus CLI.

In this blog post, I'll introduce GitHub Actions and share a concrete example of how you can get started with them to push build artifacts to Octopus, create a release and deploy it to a development environment.

## What are GitHub Actions? 

[GitHub Actions](https://docs.github.com/en/actions) are a popular new platform to automate software development workflows like CI/CD build around the GitHub ecosystem. Your define your workflow using a YAML configuration file and store it within your Git repository. You can compose your automation with resusable building blocks called actions. Workflows are executed in containers for a repeatable and reliable process.

Below is an example GitHub Action job workflow to build a simple .NET web application. 

```yaml
name: Build

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Setup .NET
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: 5.0.x
    - name: Restore dependencies
      run: dotnet restore
    - name: Build
      run: dotnet build --no-restore
    - name: Test
      run: dotnet test --no-build --verbosity normal
```

The workflow is named build and it triggers a single job whenever a push or pull request is created for the `master` branch. The steps are straightforward in that they prepare and execute a continous integration build including restoring dependencies, executing the build and running all tests. 

I highly recommend this introduction to GitHub actions to learn more. This blog post assumes you are familiar with the basics of building a workflow with GitHub actions.

## Getting started with the GitHub Actions for Octopus Deploy

To illustrate how to use the new GitHub Actions for Octopus, we'll update the example build script to 

### Install the Octopus CLI



### Push build artifacts to Octopus



### Create a release and deploy it to the DEV environment



## Conclusion

GitHub Actions for Octopus Deploy are now available. This release includes actions to install the Octopus CLI, push packages to an Octopus instance and support to create and deploy releases as well as execute runbooks. You can now automate your builds with GitHub Actions and integrate with Octopus for all your deployment and runbook automation needs.

