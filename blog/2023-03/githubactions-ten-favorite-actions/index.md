---
title: 10 of our favorite actions for GitHub Actions
description: GitHub Actions is a newcomer to Continuous Integration and provides CI as a Service. Here are 10 of our favorite actions to install from the GitHub Marketplace.
author: andrew.corrigan@octopus.com
visibility: public
published: 2023-03-15-1400
metaImage: blogimage-githubconfigurationtop10plugins-2022.png
bannerImage: blogimage-githubconfigurationtop10plugins-2022.png
bannerImageAlt: Blue plug in a white and purple socket.
isFeatured: false
tags:
  - DevOps
  - CI Series
  - Continuous Integration
  - GitHub Actions
  - Testing
---

Although relatively new to the world of continuous integration (CI), GitHub’s adding of ‘Actions’ has seen its strong community build useful tasks that plug right into your repository.

Actions let you run non-standard tasks to help you test, build, and push your work to your deployment tools.

In no particular order, here are 10 of our favorites, plus [how to install them](#install-actions).

## 1: Test reporter

Showing all your test results in GitHub, the [test reporter](https://github.com/marketplace/actions/test-reporter) action helps keep the important parts of your code and testing processes in one place. Providing the results in XML or JSON formats as part of a ‘check run’, this action tells you where your code failed with useful stats.

Test reporter already supports most of the popular testing tools for the likes of .NET, JavaScript and more. Plus, you can add more by raising an issue or contributing yourself.

Supported frameworks:

-	.NET: xUnit, NUnit, and MSTest
-	Dart: test
-	Flutter: test
-	Java: JUnit
-	JavaScript: JEST and Mocha

## 2: Build and push docker images

Doing what the title says, the [build and push Docker images action](https://github.com/marketplace/actions/build-and-push-docker-images) lets you build and push Docker images.

Using [Buildx](https://github.com/docker/buildx) and [Moby BuildKit](https://github.com/moby/buildkit) features, you can create multi-platform builds, test images, customize inputs and outputs, and more.

Check out the action’s page for the full list of features, including [advanced use](https://github.com/marketplace/actions/build-and-push-docker-images#advanced-usage) and how to [customize it](https://github.com/marketplace/actions/build-and-push-docker-images#customizing).

## 3: Setup PHP

The [Setup PHP action](https://github.com/marketplace/actions/setup-php-action) allows you to setup PHP extensions and .ini files for application testing on all major operating systems.

It’s also compatible with tools like GitHub’s composer, PHP-config, symfony, and more. See the marketplace page for the [full list of compatible tools](https://github.com/marketplace/actions/setup-php-action#wrench-tools-support).

## 4: GitTools actions {#git-tools}

The [GitTools Action](https://github.com/marketplace/actions/gittools) allows you to use both [GitVersion](https://gitversion.net/) and [GitReleaseManager](https://github.com/GitTools/GitReleaseManager) in your pipeline.

GitVersion helps solve common versioning problems with semantic versioning (also known as ‘Semver’), for consistency across your projects. GitVersion helps avoid duplication, saves rebuilding time, and much more. Benefiting CI, it creates version numbers that labels builds and makes variables available to the rest of your pipeline.

Meanwhile, GitReleaseManager automatically creates, attaches, and publishes exportable release notes.

If you only need the versioning of GitVersion, there is an [alternative action with the same name](#gitversion-action) later in this list.

## 5: Action automatic releases

Once set to react to the GitHub events of your choosing (such as commits to your main branch), the [action automatic releases](https://github.com/marketplace/actions/automatic-releases) workflow can:

-	Auto-upload assets
-	Create changelogs
-	Create a new release
-	Set the project to ‘pre-release’

## 6: Repository dispatch

The [repository dispatch action](https://github.com/marketplace/actions/repository-dispatch) makes it easier to trigger actions from a 'repository dispatch' event. Plus, it lets you trigger and chain the actions from one or more repositories.

You need to [create a personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) for this action to work as GitHub won’t support it by default.

## 7: PullPreview

The [PullPreview action](https://github.com/marketplace/actions/pullpreview) allows you to preview pull requests by spinning up live environments for code reviews. 

When making a pull with the ‘pullpreview’ label, this action checks out your code and deploys to an AWS Lightsail instance with Docker and Docker Compose.

This allows you to play with your new features as your customers would, or to show off working models of your ideas.

It also promises compatibility with your existing tools and full integration with GitHub.

The only thing you should be aware of, is that you need to buy a license if using it with commercial products.

## 8: ReportGenerator

The [ReportGenerator action](https://github.com/marketplace/actions/reportgenerator) can extract the most useful parts of coverage reports into easier to read formats. It allows you to read your data in the likes of HTML, XML, plus various text summaries and language-specific formats.

## 9: Git version {#gitversion-action}

While a little like the [GitVersion ***tool***](#git-tools) enabled by the GitTools action, this [Git version](https://github.com/marketplace/actions/git-version) is an action itself.

Like the external tool, however, it offers simple Semver versioning to help track your different releases. This is useful if you only want help versioning and don’t need GitReleaseManager.

## 10: GitHub Action tester (github-action-tester)

The [github-action-tester](https://github.com/marketplace/actions/github-action-tester) is an action that lets you kick off shell scripts for testing.

After it's installed, just add your scripts to your repository and kick them off with the events you need.

## Bonus round: Octopus

As an official GitHub technology partner, Octopus Deploy has [10 verified Octopus actions in the GitHub Marketplace](https://github.com/marketplace?query=octopus&type=actions&verification=verified_creator) that make it easy to automate your deployment processes, execute tasks, create packages, and more.

Given the nature of GitHub Actions as a service, other users have [contributed some Octopus-related actions](https://github.com/marketplace?type=&verification=&query=Octopus+) too. Check those out if you’re after even more integration with Octopus. You can also learn why we recommend you [build with GitHub and deploy with Octopus](https://octopus.com/github).

## How to install actions {#install-actions}

Installing actions in GitHub is simple:

1. Find the action you want on the [GitHub Marketplace](https://github.com/marketplace?type=actions).
2. Read the Marketplace page to check for prerequisites.
3. Click **Use latest version** in the top right (or select an older version if you need).
4. Copy the code from the pop-up, paste it into your repository’s .yml file, and save.
5. Make sure you read the action’s documentation to check for any extra setup and how to use the action.

## What next?

If the actions we chose don’t suit your project or you need something outside the scope of CI, there's plenty more to choose from. Search through the [GitHub marketplace](https://github.com/marketplace?type=actions) for more.

!include <github-actions-free-tool>

You can also learn more about how [Octopus takes your GitHub deployments to the next level](https://octopus.com/github).

Happy deployments!