---
title: "What's New in GitHub Actions for Octopus Deploy v3"
description: "We have shipped the third version of GitHub Actions for Octopus Deploy with new features. Learn more."
author: shannon.lewis@octopus.com
visibility: private
published: 9999-12-01-1400
metaImage: github-actions-for-octopus-deploy-v3.png
bannerImage: github-actions-for-octopus-deploy-v3.png
bannerImageAlt:
tags:
  - Product
  - DevOps
  - GitHub Actions
  - Integrations
---

# What's New in GitHub Actions for Octopus Deploy v3

We published our first set of [GitHub Actions to the GitHub Marketplace in June 2021](https://octopus.com/blog/github-actions-for-octopus-deploy). Then in September 2022, [we updated these actions to include many new features](https://octopus.com/blog/new-in-github-actions). Today, I am happy to announce the third iteration of GitHub Actions for Octopus Deploy. In this blog post, we're going to do a technical deep dive into the key changes that come with this release.

First up, some keys points of interest:

- We have eliminated the dependency on the Octopus CLI
- [install-octopus-cli-action](https://github.com/marketplace/actions/install-octopus-cli) now installs the [our Go-based CLI (`octopus`)](https://github.com/OctopusDeploy/cli)
- Related to the first change, the standard environment variable names have changed
- New actions for deploying releases and executing runbooks; [deploy-release-action](https://github.com/marketplace/actions/deploy-release-action), [deploy-release-tenanted-action](https://github.com/marketplace/actions/deploy-release-tenanted-action), and [await-task-action](https://github.com/marketplace/actions/await-task-action)
- New actions for creating Zip and NuGet packages; [create-zip-package-action](https://github.com/marketplace/actions/create-zip-package-action) and [create-nuget-package-action](https://github.com/marketplace/actions/create-nuget-package-action)
- Chaining actions is a built-in feature

Before we get into some examples of using the new GitHub Actions, let's talk about these changes and a bit about why we made them.

## Octopus CLI is No Longer Required

This is the big architectural change that we've made to our GitHub Actions. They no longer use the Octopus CLI to perform work; instead, they interact with the Octopus API directly from TypeScript. This means your workflows will initialize and execute far faster than before.

**This does not mean you cannot use the Octopus CLI** -- rather, you are no longer required to include the `install-octopus-cli-action` in your workflow if you only need to use our other actions. The `install-octopus-cli-action` is available for you to use if you have a script of your own that needs to use it.

## Install Octopus CLI Action Now Installs the Go-Based CLI

We've recently invested in moving our CLI implementation from C# to Go (for the reasons and details on this, please see [Building Octopus CLI vNext](https://octopus.com/blog/building-octopus-cli-vnext)). The Octopus CLI (`octo`) will remain supported until mid-2023. In fact, v1 of the `install-octopus-cli-action` will continue to install the Octopus CLI (`octo`). If you have existing workflows using the C#-based CLI, you can continue to use v1 of this action.

`install-octopus-cli-action` v3 (or greater) will only install the new Go-based CLI (`octopus`). If you are writing new workflows, we would strongly recommend using v3. The Go-based CLI (`octopus`) has a number of new features and improvements over the C#-based CLI. That stated, there are some minor differences. If necessary, these CLIs can be used side-by-side.

## Environment Variables Names

We've been [advocating using environment variables, rather than the CLI parameters, in the actions](https://octopus.com/blog/new-in-github-actions#improved-usability) as this brings with it a number of security benefits.

We still very much encourage you to use environment variables for setting sensitive values (i.e. API keys) in the new version of the actions the names have changed, because it's no longer the Octopus CLI picking them up.

| Value              | Old Variable          | New Variable      |
| :----------------- | :-------------------- | :---------------- |
| Octopus Server URL | `OCTOPUS_CLI_SERVER`  | `OCTOPUS_URL`     |
| Octopus API Key    | `OCTOPUS_CLI_API_KEY` | `OCTOPUS_API_KEY` |
| Octopus Space Name |                       | `OCTOPUS_SPACE`   |

## Deployment and Runbook Run Actions

GitHub Actions for Octopus Deploy v3 introduces three new actions for deployment and runbook runs:

- [deploy-release-action](https://github.com/marketplace/actions/deploy-release-action)
- [deploy-release-tenanted-action](https://github.com/marketplace/actions/deploy-release-tenanted-action)
- [await-task-action](https://github.com/marketplace/actions/await-task-action)

In v1 of the `create-release-action` we supported the old `deploy-to` parameter from the Octopus CLI (`octo`). Unfortunately, this also brought with it all of the other deployment-related switches that the Octopus CLI (`octo`) supports. This bloated the action parameters, making them complex and confusing. As an example, the `--variable` parameter often tripped people up. It only works for setting the value of a prompted variable for a deployment, but it looked like it could be used for setting a project variable value during release creation.

Based on ongoing issues being experienced with these parameters, we made the decision in GitHub Actions for Octopus Deploy v2 to drop them and remove the confusion. The longer term goal was what we now have in v3, separate actions for queuing deployments (and runbooks runs).

Something else the Octopus CLI (`octo`) did was to bundle the concept of whether to wait for the deployment to complete into the same command. We've separated into its own action too. This might seem overkill at first, but when used in conjunction with some features of GitHub Actions allows greater flexibility. We'll talk about this in more details in the examples below.

NOTE: tenanted deployments have different semantics to "standard" deployments. Primarily, they support a different multiplicity on the environments you can deploy to (standard can deploy to multiple environments, tenanted can only be to a single environment). To make this clear in the action contracts `deploy-release-tenanted-action` is separate to `deploy-release-action`.

NOTE: while this is the initial version of these actions, we decided to release them as v3 to make it easier to reason about these new actions as a matching set. The versions will diverge again over time as we move forward and make patches/updates to the actions individually.

# Actions for Creating Zip and NuGet Packages

GitHub Actions for Octopus Deploy v3 introduces two new actions for package creation:

- [create-zip-package-action](https://github.com/marketplace/actions/create-zip-package-action)
- [create-nuget-package-action](https://github.com/marketplace/actions/create-nuget-package-action)

Zip and NuGet packages are archive files that are used to distribute and deploy software applications. Zip is a widely-used archive format that can be opened by many different applications. NuGet packages, on the other hand, are specifically designed for use with the Microsoft development platform, and are used to distribute libraries and other resources that can be easily integrated into .NET applications. Both Zip and NuGet packages are commonly used to distribute software because they provide a convenient and efficient way to package and distribute large numbers of files. Typically, we observed customers using the Octopus CLI (`octo`) to generate packages through the `pack` command. These actions eliminate this requirement whilst providing an integrated experience through GitHub Actions.

## Chaining is a Build-In Feature

A number of the actions produce outputs, to enable chaining of the actions. The outputs are as follows:

| action                           | outputs                  | description                                                                    |
| :------------------------------- | :----------------------- | :----------------------------------------------------------------------------- |
| `create-release-action`          | `release_number`         | The release number (version) that was created                                  |
| `deploy-release-action`          | `server_tasks`           | JSON array of objects with `serverTaskId` and `environmentName`                |
| `deploy-release-tenanted-action` | `server_tasks`           | JSON array of objects with `serverTaskId` and `tenantName`                     |
| `run-runbook-action`             | `server_tasks`           | JSON array of objects with `serverTaskId`, `environmentName`, and `tenantName` |
| `await-task-action`              | `completed_successfully` | Whether the task completed successfully or not                                 |

We'll show more about how to use the JSON arrays in the examples below.

On the output of the `await-task-action`, note that the action will fail if the task doesn't complete successfully. You can then use the [step's outcome](https://docs.github.com/en/actions/learn-github-actions/contexts#steps-context) in conjunction with the `completed_successfully` if you want to take different action in the workflow based on whether the deployment/run failed versus some other failure occurred (like comms was lost to the Octopus instance or something like that).

# Common Workflow Patterns

## All-in-One

This is the pattern we see most widely used today, driven by the nature of the way the CLI encourages you to do things. It centres around all actions being steps in the one job:

```yaml
- name: Create Zip package 🐙
  id: package
  uses: OctopusDeploy/create-zip-package-action@v3
  with:
    package_id: DemoNetCoreWebAppGHASingleJob
    version: ${{ steps.build.outputs.version }}
    base_path: ${{ steps.build.outputs.output_folder }}
    files: "**/*"
    output_folder: packaged

- uses: actions/upload-artifact@v3
  with:
  name: ${{ steps.package.outputs.package_filename }}
  path: ${{ steps.package.outputs.package_file_path }}

- name: Push a package to Octopus Deploy 🐙
  uses: OctopusDeploy/push-package-action@v3
  with:
    packages: ${{ steps.package.outputs.package_file_path }}

- name: Push build information to Octopus Deploy 🐙
  uses: OctopusDeploy/push-build-information-action@v3
  with:
    version: ${{ steps.build.outputs.version }}
    packages: MyPackage

- name: Create a release in Octopus Deploy 🐙
  uses: OctopusDeploy/create-release-action@v3
  id: "create_release"
  with:
    project: "Pet Shop"
    package_version: ${{ steps.build.outputs.version }}

- name: Deploy the release in Octopus Deploy 🐙
  uses: OctopusDeploy/deploy-release-action@v3
  id: "queue_deployments"
  with:
    project: "Pet Shop"
    release_number: ${{ steps.create_release.outputs.release_number }}
    environments: |
      Development
      Integration

- name: Waiting for 1st deployment in Octopus Deploy 🐙
  uses: OctopusDeploy/await-task-action@v3
  with:
    server_task_id: ${{ fromJson(steps.queue_deployments.outputs.server_tasks)[0].serverTaskId }}

- name: Waiting for 2nd deployment in Octopus Deploy 🐙
  uses: OctopusDeploy/await-task-action@v3
  with:
    server_task_id: ${{ fromJson(steps.queue_deployments.outputs.server_tasks)[1].serverTaskId }}
```

The output of that workflow running in GitHub Actions would look something like this:

![Single Job](single-job-example.png)

Using this pattern provides the following benefits:

- Steps are relatively easy to chain together in the workflow file (compared to what you'll see in the next example)
- Suitable for deploying to a single environment/tenant

This pattern has the following disadvantages:

- Once there are multiples, you have no guarantee on the order they are queued
- You cannot see at a glance which task is for which environment
- Steps execute serially, so the wait for the second deployment doesn't start waiting until the wait for the first is complete

## Separate the Actions into Multiple Jobs

In this example, we illustrate a workflow that uses multiple jobs to coordinate the actions. It utilises both jobs and a feature in GitHub Actions called [matrixes](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs):

```yaml
jobs:
  build:
    name: Build and unit test code
    runs-on: ubuntu-latest

    outputs:
      version: ${{ steps.build.outputs.version }}
      artifact_name: ${{ steps.package.outputs.package_filename }}

    steps:
      - uses: actions/checkout@v3

      - name: Setup .NET
        uses: actions/setup-dotnet@v2
        with:
          dotnet-version: 6.0.x

      - name: Run Build 🏗
        id: build
        run: |
          # do whatever you do to build your package. Assume this outputs a version variable and the folder where it produced output

      - name: Create Zip package 🐙
        id: package
        uses: OctopusDeploy/create-zip-package-action@v3
        with:
          package_id: MyPackage
          version: ${{ steps.build.outputs.version }}
          base_path: ${{ steps.build.outputs.output_folder }}
          files: "**/*"
          output_folder: packaged

      - uses: actions/upload-artifact@v3
        with:
          name: ${{ steps.package.outputs.package_filename }}
          path: ${{ steps.package.outputs.package_file_path }}

  push:
    name: Push information to Octopus
    needs: build
    runs-on: ubuntu-latest

    env:
      OCTOPUS_URL: ${{ secrets.OCTOPUS_URL }}
      OCTOPUS_API_KEY: ${{ secrets.OCTOPUS_API_KEY }}
      OCTOPUS_SPACE: "Galaxy"

    steps:
      - uses: actions/download-artifact@v3
        with:
          name: MyPackage.${{ needs.build.outputs.version }}
          path: package

      - name: Push a package to Octopus Deploy 🐙
        uses: OctopusDeploy/push-package-action@v3
        with:
          packages: package/MyPackage.${{ needs.build.outputs.version }}.zip

      - name: Push build information to Octopus Deploy 🐙
        uses: OctopusDeploy/push-build-information-action@v3
        with:
          version: ${{ needs.build.outputs.version }}
          packages: MyPackage

  snapshot:
    name: Snapshot information in Octopus
    needs: [build, push]
    runs-on: ubuntu-latest

    outputs:
      release_number: ${{ steps.create_release.outputs.release_number }}

    env:
      OCTOPUS_URL: ${{ secrets.OCTOPUS_URL }}
      OCTOPUS_API_KEY: ${{ secrets.OCTOPUS_API_KEY }}
      OCTOPUS_SPACE: "Galaxy"

    steps:
      - name: Create a release in Octopus Deploy 🐙
        id: "create_release"
        uses: OctopusDeploy/create-release-action@v3
        with:
          project: "Rockets"
          package_version: ${{ needs.build.outputs.version }}

  deploy:
    name: Deploy snapshot using Octopus
    needs: [build, snapshot]
    runs-on: ubuntu-latest

    env:
      OCTOPUS_URL: ${{ secrets.OCTOPUS_URL }}
      OCTOPUS_API_KEY: ${{ secrets.OCTOPUS_API_KEY }}
      OCTOPUS_SPACE: "Galaxy"

    outputs:
      server_tasks: ${{ steps.queue_deployments.outputs.server_tasks }}

    steps:
      - name: Deploy the release in Octopus Deploy 🐙
        uses: OctopusDeploy/deploy-release-tenanted-action@v3
        id: "queue_deployments"
        with:
          project: "Rockets"
          release_number: ${{ needs.snapshot.outputs.release_number }}
          environment: Development
          tenants: Mars
          tenant_tags: |
            planets/gas giants

  wait:
    needs: deploy
    runs-on: ubuntu-latest
    name: ${{ matrix.deployment.tenantName }}

    env:
      OCTOPUS_URL: ${{ secrets.OCTOPUS_URL }}
      OCTOPUS_API_KEY: ${{ secrets.OCTOPUS_API_KEY }}
      OCTOPUS_SPACE: "Galaxy"

    strategy:
      matrix:
        deployment: ${{ fromJson(needs.deploy.outputs.server_tasks) }}

    steps:
      - name: Waiting for deployment in Octopus Deploy 🐙
        uses: OctopusDeploy/await-task-action@enh-initialversion
        with:
          server_task_id: ${{ matrix.deployment.serverTaskId }}
```

The output of that workflow running in GitHub Actions would look something like this:

![Multiple Jobs](multiple-jobs-example.png)

Using this pattern provides the following benefits:

- When there are multiple deployments involved, it is really easy to determine which is which
- Waiting for the tasks happens in parallel (this is the matrix feature in GitHub Actions comes into play)
- Tasks appear in the expandable section on the diagram but also as individual entries in the summary on the left (again, matrixes coming into play)
- Individual jobs within the workflow can be rerun if they fail. For example, imagine the packages got built and pushed to Octopus but the release creation failed because of a config error in Octopus. If you corrected the config you can then rerun the workflow from the same point without having to rebuild the packages and push them again (both of which can be potentially expensive operations)

This pattern has the following disadvantages:

- The workflow is more complex to setup. Passing outputs from steps across job boundaries requires a bit more ceremony in the yaml
- The matrix can feel like overkill if there is only a single deployment

## Runbook Runs

Executing runbooks is really not that different to deploying releases. We're not going to provide a full example here but wanted to call out one specific aspect of the output data that action provides and what that means for the matrix configuration.

In the previous example, there is this entry to bind the name of the matrix job to the tenant's name from the JSON output data.

```yaml
name: ${{ matrix.deployment.tenantName }}
```

With runbook runs the output data contains both the `tenantName` and the `environmentName`, because it allows you to request multiples of both at once and will execute for any matching tenants that have a connection to the given project, for the given environments. What this means is that your matrix name can do something like this

```yaml
name: ${{ matrix.deployment.tenantName }} - ${{ matrix.deployment.environmentName }}
```

or

```yaml
name: ${{ matrix.deployment.environmentName }} - ${{ matrix.deployment.tenantName }}
```

depending on which value is more important to you. This can be important because in the summary view the GitHub Actions UI will truncate long values, so having the most important information up front makes it easier to find at a glance (you can always see the full details once you drill in, it just takes extra clicking).

# Summary

GitHub Actions for Octopus Deploy v3 offers a significant improvement to the previous version, with an additional five actions added to the product lineup. These new actions enhance the ability to automate deployment processes, execute tasks, and create packages. They also greatly improve the overall user experience. We use these actions ourselves, which is a testament to their effectiveness and reliability. This latest release showcases our commitment to providing powerful, user-friendly actions for managing your GitHub deployments.
