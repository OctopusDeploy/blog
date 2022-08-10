---
title: Announcing GitHub Actions for Octopus Deploy
description: You can now integrate your GitHub builds with deployments in Octopus Deploy thanks to our GitHub Actions.
author: rob.pearson@octopus.com
visibility: public
published: 2021-06-02-1400
metaImage: github-actions-integration.png
bannerImage: github-actions-integration.png
bannerImageAlt: GitHub actions integrating with Octopus Deploy build
tags:
 - DevOps
 - GitHub Actions
---

![GitHub actions integrating with Octopus Deploy build](github-actions-integration.png)

We've shipped our first official GitHub Actions for Octopus Deploy. These initial actions cover the core integration scenarios to connect your GitHub builds with your deploys and runbook runs in Octopus: 

- [Push packages](https://github.com/marketplace/actions/push-package-to-octopus-deploy) to Octopus Deploy. 
- [Create releases](https://github.com/marketplace/actions/create-release-in-octopus-deploy) in Octopus Deploy.
- [Execute a runbook](https://github.com/marketplace/actions/run-runbook-in-octopus-deploy) in Octopus Deploy.
- [Install](https://github.com/marketplace/actions/install-octopus-cli) the Octopus CLI.

We plan to add additional GitHub Actions later in the year. 

In this blog post, I demonstrate how you can get started with GitHub Actions to push build artifacts to Octopus, create a release, and deploy it to a development environment.

## What is GitHub Actions? 

[GitHub Actions](https://docs.github.com/en/actions) is a popular new platform to automate software development workflows, like CI/CD built around the GitHub ecosystem. 

You define your workflow using a YAML configuration file and store it in your Git repository. You can compose your automation with reusable building blocks called actions. Workflows are executed in containers for a repeatable and reliable process.

Below is an example GitHub Action job workflow to build a simple .NET web application. GitHub provides examples for most programming languages and frameworks.

```yaml
name: Build

on:
  push:
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
      run: dotnet publish -o build 
    - name: Test
      run: dotnet test --no-build --verbosity normal
```

The workflow is named **build** and it triggers a single job whenever changes are pushed to the parent Git repository on the `master` branch. The steps prepare and execute a continuous integration build including:

- Restoring dependencies
- Executing the build
- Running all tests

I recommend reading [GitHub's docs](https://docs.github.com/en/actions/learn-github-actions) to learn more. This blog post assumes you are familiar with the basics of building a workflow with GitHub actions.

## Getting started with GitHub Actions for Octopus Deploy

To illustrate how to use the new GitHub Actions for Octopus, we'll update the example build script above to install the Octopus CLI, package and push our build artifact to Octopus and then create a release and deploy it to our dev environment.

The complete workflow file is available in the [GitHub Samples repository](https://github.com/OctopusSamples/RandomQuotes/actions/workflows/dotnet.yml):

```yaml
name: Build

on:
  push:
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
      run: dotnet publish -o build 
    - name: Test
      run: dotnet test --no-build --verbosity normal
    - name: Install Octopus CLI 🐙
      uses: OctopusDeploy/install-octopus-cli-action@v1.1.6
      with:
        version: latest
    - name: Package build artifacts
      run: octo pack --id="RandomQuotes" --format="zip" --version="1.0.${{github.run_number}}" --basePath="/home/runner/work/RandomQuotes/RandomQuotes/build/"
    - name: Push packages to Octopus Deploy 🐙
      uses: OctopusDeploy/push-package-action@v1.0.1
      with:
        api_key: ${{ secrets.OCTOPUS_APIKEY }}
        server: ${{ secrets.OCTOPUS_SERVER }}
        packages: "RandomQuotes.1.0.${{github.run_number}}.zip"
    - name: Create a release in Octopus Deploy 🐙
      uses: OctopusDeploy/create-release-action@v1.0.2
      with:
        api_key: ${{ secrets.OCTOPUS_APIKEY }}
        server: ${{ secrets.OCTOPUS_SERVER }}
        project: "Projects-141"
        deploy_to: "Dev"

```

![GitHub Actions Secrets for the Octopus Server URL and an API key](github-action-secrets.png)

Note that we reference two secrets in this configuration. One is for the Octopus Server URL and the other is an API key to authenticate and integrate with our Octopus instance. In this case, I'm using an [Octopus Cloud instance](https://octopus.com/docs/octopus-cloud), however, you can also connect to a [self-hosted Octopus instance](https://octopus.com/docs/getting-started#self-hosted-octopus) if it's publicly accessible.


:::hint
NOTE: This is building a Microsoft .NET 5 web application but it could easily be a Spring (Java) web app or NodeJS express service, etc. The important parts are how the GitHub Actions for Octopus make it easy to integrate.
:::

### Installing the Octopus CLI

```yaml
    - name: Install Octopus CLI 🐙
      uses: OctopusDeploy/install-octopus-cli-action@v1.1.6
      with:
        version: latest
```

To integrate with an Octopus Server, first install the Octopus CLI. This is a pre-requisite to use any of the other steps as it bootstraps the job runner with the appropriate dependencies to install the Octopus CLI.

### Pushing build artifacts to Octopus

```yaml
    - name: Pack
      run: octo pack --id="RandomQuotes" --format="zip" --version="1.0.${{github.run_number}}" --basePath="/home/runner/work/RandomQuotes/RandomQuotes/build/" --verbose
    - name: Push a package to Octopus Deploy 🐙
      uses: OctopusDeploy/push-package-action@v1.0.1
      with:
        api_key: ${{ secrets.OCTOPUS_APIKEY }}
        server: ${{ secrets.OCTOPUS_SERVER }}
        packages: "RandomQuotes.1.0.${{github.run_number}}.zip"
```

The next step is packaging your build artifacts and pushing them to a package repository. In this case, we're pushing to the Octopus built-in package repository which is a popular option. 

There are two steps to package and push my build artifacts:

1. Packaging my build output as a ZIP file.
1. Pushing the package to my Octopus instance. 

As mentioned, I'm referencing two secrets stored in my repository configuration. One is for the Octopus Server URL and the other is an API key for my GitHub build. 

### Creating a release and deploying it to the dev environment

```yaml
    - name: Create a release in Octopus Deploy 🐙
      uses: OctopusDeploy/create-release-action@v1.0.2
      with:
        api_key: ${{ secrets.OCTOPUS_APIKEY }}
        server: ${{ secrets.OCTOPUS_SERVER }}
        project: "Projects-141"
        deploy_to: "Dev"
```

The final step in my build process is to create a release of my project and deploy it to my dev environment. This is done with a single step; I supply my project ID and the environment name that I want to deploy to. And that's it.

### Success!

![Successful GitHub build](github-build.png)

If we push a commit to our repository, we can see the GitHub Action run and its output. It might take a few iterations to fix syntax and get everything right, but the outcome is a successful build. 

## Conclusion

GitHub Actions for Octopus Deploy is now available. This release includes actions to install the Octopus CLI and push packages to an Octopus instance, plus support to create and deploy releases and execute runbooks. 

You can now automate your builds with GitHub Actions and integrate with Octopus for all your deployment and runbook automation needs.

We plan to add additional actions for Octopus. Please let us know in the comments if there are any you'd like us to add. 

Happy deployments!
