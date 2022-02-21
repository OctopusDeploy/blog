---
title: How GitHub Actions is different to traditional build servers
description: We explore how GitHub Actions, providing Continuous Integration as a Service, is different to traditional build servers.
author: andrew.corrigan@octopus.com
visibility: public
published: 2022-02-28-1400
metaImage: blogimage-howgithubactionsidifferenttoatraditionalciserver-2022.png
bannerImage: blogimage-howgithubactionsidifferenttoatraditionalciserver-2022.png
bannerImageAlt: Two blue Octopus tentacles, one holding a red apple, and one holding a green apple.
isFeatured: false
tags:
  - DevOps
  - CI Series
  - Continuous Integration
  - GitHub Actions
---

GitHub Actions is relatively new to the world of automation and Continuous Integration (CI). Providing ‘CI as a Service’, GitHub Actions has many differences to its traditional, rival platforms.

In this post, we explore the differences between GitHub Actions and traditional build severs. We also look at whether GitHub Actions is a suitable option for building and testing your code.


## GitHub Actions is CI already built into GitHub

One of the biggest advantages GitHub Actions has over traditional CI platforms is its delivery. GitHub Actions is ‘CI as a Service’, and already included with every repository for all customers (more on the pricing shortly). 

Simply put: if you have an account with GitHub, you can use GitHub Actions.

This is appealing if you like all your work in one place, with fewer moving parts to worry about.

## Actions isn't an option if you don't use GitHub

Your ability to use GitHub Actions depends on you or your company using GitHub. If you use other code repositories, or you host your code on-premises, GitHub Actions isn’t an option.

It’s worth mentioning, however, that other code repository services, like GitLab and BitBucket, have released their own takes on CI as a Service, too. You may still have options if other services are more your flavor.

## No hardware needed for CI (unless you want to)

By default, actions use ‘runners’ to complete a workflow’s jobs. Runners are GitHub-hosted virtual machines using Windows, Linux, and MacOS. This means you don’t need to maintain your own infrastructure to perform CI. There’s no hardware to house, no operating systems to maintain, no installs, updates, or patches. GitHub takes care of all this for you.

You can also [self-host your own runners](https://docs.github.com/en/actions/hosting-your-own-runners) if you need specific setups for CI and GitHub’s runners aren’t up to the task, or you want more control.

## Licensing and pricing

Traditional build servers are usually open source or licensed to scale with your needs, by number of users or instances. CI as a Service, however, means providers can get creative with their pricing models.

GitHub Actions, for example, [adopts a pay-as-you-go approach](https://docs.github.com/en/billing/managing-billing-for-github-actions/about-billing-for-github-actions) and measures all jobs by the minutes they take to run.

All users get free monthly minutes and storage for jobs, with the numbers scaled alongside your plan. If you exceed your free minutes, you get billed per-minute, weighted against the operating system the runner needs for the job.

GitHub Actions is entirely free for those with public repos or using self-hosted runners.

## How GitHub Actions automates with events and workflows

GitHub Actions uses activity in your GitHub repo (or an external event, if you use a ‘[repository dispatch event](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#repository_dispatch)’ webhook) to trigger workflows. You can choose to start workflows with single or several events, use a schedule, or kick them off manually.

The structure of a workflow is as follows: 

- Each workflow contains jobs
- Each job has steps it performs on its own runner
- A step is an individual action in each job

For example, you could create a workflow that:

1. Tests your code whenever someone creates a pull request
1. Packages the code after successful tests and pushes it to your deployment tool

With this scenario, the workflow would have 2 separate jobs. If testing on job one is successful, that will trigger job 2. Both jobs have their own steps and run on separate, clean runners.

Thanks to the [GitHub Marketplace](https://github.com/marketplace), you might not need to create the workflows at all. Though not unique to GitHub Actions, there are thousands of community-made actions and workflows you can use to achieve what you need.

Octopus has also created a [tool to help you build GitHub Actions workflows](https://githubactionworkflows.com/). Try it out if you're unsure where to start.

For more information on workflows in GitHub Actions, check out:

- [Understanding GitHub Actions](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions)
- [The full list of GitHub events you can use to trigger workflows](https://docs.github.com/en/actions/learn-github-actions/events-that-trigger-workflows)
- [Workflow syntax for GitHub Actions](https://docs.github.com/en/actions/learn-github-actions/workflow-syntax-for-github-actions)

## How you install actions

Installing actions is also different compared to the way you add and manage plugins in most other CI platforms.

In Jenkins, for example, they’re almost modular mini applications. Installed using .hpi packages, a Jenkins plugin can drastically affect the look, feel, and the options you see in your Jenkins instance.

Installing actions in GitHub involves copying code from an action’s marketplace page and pasting it into your repository’s .yml file. Instead of providing visible changes, actions offer only new functionality, triggered by events from the action’s own repo.

## Conclusion

This post covered the major differences between GitHub Actions (and the concept of CI as a Service) and traditional CI platforms. 

GitHub Actions is an ideal solution if you:

- Are new to the concepts of CI
- Don’t want to spend time setting up and maintaining a dedicated build server
- Don’t want or need to host your hardware or data on-premises
- Like all your tools in one place
- Already use GitHub

!include <q1-2022-newsletter-cta>

Happy deployments!
