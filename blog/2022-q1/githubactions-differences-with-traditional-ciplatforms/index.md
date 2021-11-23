---
title: How GitHub Actions is different to traditional build servers
description: We explore how GitHub Actions, providing 'Continuous integration as a service', is different to traditional build servers
author: Andrew.Corrigan@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags:
  - DevOps
  - Continuous Integration
  - Jenkins
  - GitHub Actions
  - Testing
---

GitHub Actions is a relative newcomer to the world of automation and Continuous Integration (CI). Providing ‘CI as a Service’, GitHub Actions has many differences to its traditional, rival platforms.

In this blog we explore those differences, looking at why they might make GitHub Actions a suitable option (or not) for building and testing your code.


## Actions is CI already built into GitHub

One of the biggest advantages GitHub Actions has over traditional CI platforms is its delivery. GitHub Actions is ‘CI as a Service’, and already included with every repository and for all customers (more on the pricing shortly). 

Simply put: if you have an account with GitHub, you can use GitHub Actions.

This is appealing for those that like all their work in one place, meaning you have fewer moving parts to worry about.

## Actions is not an option if GitHub isn’t your code repository

It’s kind of obvious, but your ability to use GitHub Actions depends entirely on you or your company using GitHub. If you use other code repositories, or you host your code on-premises, GitHub Actions isn’t an option.

That said, it’s worth mentioning that other code repository services like GitLab and BitBucket have released their own takes on CI as a Service too. You may still have options if other services are more your flavour.

## You don’t need your own hardware for CI (unless you really want to)

By default, actions use ‘runners’ to complete the workflow’s jobs. Runners are GitHub-hosted virtual machines using Windows, Linux and MacOS. This means you don’t need to maintain your own infrastructure to perform CI. There’s no hardware to house, no operating systems to maintain, no installs, updates, or patches. GitHub takes care of all this for you.

That said, you can [self-host your own runners](https://docs.github.com/en/actions/hosting-your-own-runners) if you need specific setups for CI and GitHub’s runners aren’t up to the task or you want more control.

##Licensing and pricing

Traditional build servers are usually open source or licensed to scale with your needs, be that number of users or instances. CI as a Service, however, means providers can get creative with their pricing models.

GitHub Actions, for example, [adopts a pay-as-you-go approach](https://docs.github.com/en/billing/managing-billing-for-github-actions/about-billing-for-github-actions) and measures all jobs by the minutes they take to run.

All users get free monthly minutes and storage for jobs, with the numbers scaled alongside your plan. If you exceed your free minutes, you get billed per-minute, weighted against the operating system the runner needs for the job.

Depending on GitHub Actions is entirely free for those with public repos or using self-hosted runners.

## How GitHub Actions automates with events and workflows

Actions uses activity in your GitHub repo (or an external event, if you use a ‘[repository dispatch event](https://rm2wdx0x6j.execute-api.us-west-1.amazonaws.com/Development/index.html)’ webhook) to trigger workflows. You could choose to start workflows with single or several events, use a schedule, or just kick them off manually.

The structure of a workflow is as follows: Each workflow contains jobs. Each job has steps it performs on its own runner. A step is an individual action in each job.

For example, you could create a workflow that:

1. tests your code whenever someone creates a pull request
2. packages the code after successful tests and pushes it to your deployment tool.

With this scenario, the workflow would have 2 separate jobs. If testing on job 1 is successful, that will trigger job 2. Both jobs have their own steps and run on separate, clean runners.

GitHub workflows are reusable and referenceable, so you can use them across projects and repos. Plus, you might not need to create them at all. Though not unique to GitHub Actions, there are thousands of community-made actions and workflows you can use to achieve what you need.

Octopus has also created a [tool to help you build GitHub Actions workflows](https://githubactionworkflows.com/). Give it a whirl if unsure where to start.

For more information on workflows in GitHub Actions, why not check out:

- [understanding GitHub Actions](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions)
- [the full list of GitHub events you can use to trigger workflows](https://docs.github.com/en/actions/learn-github-actions/events-that-trigger-workflows)
- [workflow syntax for GitHub Actions](https://docs.github.com/en/actions/learn-github-actions/workflow-syntax-for-github-actions)

## How you install actions

Installing actions is also different compared to the way you add and manage plugins in most other CI platforms.

In Jenkins, for example, they’re almost modular mini applications. Installed using .hpi packages, a Jenkins plugin can drastically affect the look, feel and the options you’ll see in your Jenkins instance.

Installing actions in GitHub involves only copying code from an action’s marketplace page and pasting it into your repository’s .yml file. Instead of providing visible changes, actions offer only new functionality, triggered by events from the action’s own repo.

## Conclusion

So, there you have the major differences between GitHub Actions (and the concept of CI as a Service) and traditional CI platforms. GitHub Actions is an ideal solution for those who:

- are new to the concepts of CI
- don’t want to spend time setting up and maintaining a dedicated build server
- don’t want or need to host their hardware or data on-premises
- like all their tools in one place
- already use GitHub.

Happy deployments!