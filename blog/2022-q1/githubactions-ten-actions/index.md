---
title: 10 of our favourite GitHub Actions
description: GitHub Actions is a newcomer to continuous integration and provides CI as a Service. Here are 10 of our favorite actions to install from the GitHub Marketplace.
author: andy.corrigan@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags:
  - DevOps
  - Continuous Integration
  - GitHub Actions
  - Testing
---

Although a relative newcomer to the world of continuous integration (CI), GitHub’s adding of ‘Actions’ has seen its already strong community build useful tasks that plug right into your repository.

These workflows allow you to run all kinds of non-standard tasks to help you test, build and push your work to your deployment tools.

In no particular order, here are 10 of our favorites, plus [how to install them](#install-actions).

## 1: Test reporter

Showing all your test results in GitHub, the [test reporter](https://github.com/marketplace/actions/test-reporter) action helps keep the important parts of your code and testing processes in one place. Providing the results in XML or JSON formats as part of a ‘check run’, this action offers clear messaging as to where your code failed and with useful stats.

Right now, test reporter already supports most of the popular testing tools for the likes of .NET, JavaScript and more. Plus, you can add more by raising an issue or contributing yourself.

Supported frameworks:

-	.NET: xUnit, NUnit and MSTest
-	Dart: test
-	Flutter: test
-	Java: JUnit
-	JavaScript: JEST and Mocha

## 2: Build and push docker images

Doing what the title says, the [build and push Docker images action](https://github.com/marketplace/actions/build-and-push-docker-images) lets you build and push Docker images.

Using [Buildx](https://github.com/docker/buildx) and [Moby BuildKit](https://github.com/moby/buildkit) features, you can create multi-platform builds, test images, customize inputs and outputs, and way more.

Check out the action’s page for the full list of features, including [advanced use](https://github.com/marketplace/actions/build-and-push-docker-images#advanced-usage) and how to [customize it](https://github.com/marketplace/actions/build-and-push-docker-images#customizing).

## 3: Setup PHP

The [Setup PHP action](https://github.com/marketplace/actions/setup-php-action) allows you to setup PHP extensions and.ini files for application testing on all major operating systems.

It’s also compatible with a heap of tools like GitHub’s composer, PHP-config, symfony and more. See the marketplace page for the [full list of compatible tools](https://github.com/marketplace/actions/setup-php-action#wrench-tools-support).

## 4: GitTools actions {#git-tools}

The [GitTools Action](https://github.com/marketplace/actions/gittools) allows you to use both [GitVersion](https://gitversion.net/) and [GitReleaseManager](https://github.com/GitTools/GitReleaseManager) in your pipeline.

GitVersion helps solve common versioning problems with semantic versioning (also known as ‘Semver’), for consistency across your projects. GitVersion helps avoid duplication, saves rebuilding time and way more. Benefiting CI, it creates version numbers that labels builds and makes variables available to the rest of your pipeline.

Meanwhile, GitReleaseManager automatically creates, attaches, and publishes exportable release notes.

If you only need the versioning of GitVersion, there is an [alternative action with the same name](#gitversion-action) later in this list.

## 5: Action automatic releases

Once set to react to the GitHub events of your choosing (such as commits to your main branch), the [action automatic releases](https://github.com/marketplace/actions/automatic-releases) workflow can:

-	auto-upload assets
-	create changelogs
-	create a new release
-	set the project to ‘pre-release’.

## 6: Repository dispatch

While GitHub Actions has a lot of events to trigger actions from, the [repository dispatch action](https://github.com/marketplace/actions/repository-dispatch) adds another!

This action adds a ‘repository dispatch’ event, from which you can trigger and chain actions from one or more repository.

You must [create a personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) for this action to work as GitHub won’t support it by default.

## 7: PullPreview

The [PullPreview action](https://github.com/marketplace/actions/pullpreview) allows you to preview pull requests by spinning up live environments for code reviews. When making a pull with the ‘pullpreview’ label, this action will check out your code and deploy to an AWS Lightsail instance with Docker and Docker Compose.

This allows you to play with your new features as your customers would engage with them, or to show off working models of your ideas.

It also promises compatibility with your existing tools and full integration with GitHub.
The only thing you should be aware of, however, is that you’ll need to buy a license if using it with commercial products.

## 8: ReportGenerator

The [ReportGenerator action](https://github.com/marketplace/actions/reportgenerator) can extract the most useful parts of coverage reports into easier to read formats. It allows you to read your data in the likes of HTML, XML, plus various text summaries and language-specific formats.

## 9: Git version {#gitversion-action}

While a little like the [GitVersion ***tool***](#git-tools) enabled by the GitTools action, this [Git version](https://github.com/marketplace/actions/git-version) is an action itself.

Like the external tool, however, it offers simple Semver versioning to help track your different releases. This is useful if you only want help versioning and don’t need GitReleaseManager.

## 10: GitHub Action tester (github-action-tester)

The [github-action-tester](https://github.com/marketplace/actions/github-action-tester) is an action that lets you kick off shell scripts for testing.

Once installed, just add your scripts to your repository and kick them off with whichever events you need.

## Bonus round: Octopus

We’ve created 4 useful plugins for those using Octopus to deploy through their environments.

- [Install Octopus CLI](https://github.com/marketplace/actions/install-octopus-cli) – lets you install the [Octopus Command Line Interface](https://octopus.com/docs/octopus-rest-api/octopus-cli) on runners (GitHub-hosted or otherwise). You can extend its use with our other 3 actions too, using the Octopus CLI to:
   - [Push packages](https://github.com/marketplace/actions/push-package-to-octopus-deploy)
   - [Create releases](https://github.com/marketplace/actions/create-release-in-octopus-deploy)
   - [Start runbooks](https://github.com/marketplace/actions/run-runbook-in-octopus-deploy)

And, given the nature of GitHub Actions as a service, other users have [contributed some Octopus-related actions too](https://github.com/marketplace?type=&verification=&query=Octopus+). Check those out if you’re after even more integration with Octopus.

## How to install actions {#install-actions}

Installing actions in GitHub is simple:

1. Find the action you want on the [GitHub Marketplace](https://github.com/marketplace?type=actions).
2. Read the marketplace page to check for prerequisites.
3. Click **Use latest version** in the top right (or select an older version if you need).
4. Copy the code from the pop-up, paste it into your repository’s .yml file and save.
5. Make sure you read the action’s documentation to check for any extra setup and how to use the action.

## What next?

If the actions we’ve chosen don’t suit your project or you need something outside the scope of CI, don’t panic! Take a quick search through the [GitHub marketplace](https://github.com/marketplace?type=actions) for more.

Happy deployments!