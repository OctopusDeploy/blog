---
title: An introduction to build servers
description: A brief overview on the benefits of build servers, focusing on Jenkins, GitHub Actions and what you can expect from us in the next few months
author: andrew.corrigan@octopus.com
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

Octopus can receive packages by whatever means you want, whether you upload them direct to our in-built package store or import them via a feed. As believers in Continuous Integration/Continuous Delivery (CI/CD), we think they should come from a build server (also known as a 'CI platform').

Given they can automate everything that happens between code commits and deployments, build servers are vital to CI as a concept. Let’s dig into exactly why we rate them so highly.

## Why build servers are important

Build servers have 3 main purposes:

- compiling committed code from your repository many times a day
- running thorough, automatic tests to validate code
- creating deployable packages and handing off to your deployment tool.

Life without a build server means time bogged down with manual processes and the needless time constraints caused by them. For example, without a build server:

- Your team will likely need to commit code before a daily deadline or during change windows.
- Once that deadline passes, no one can commit again until someone has manually created and tested a build.
- If there are problems with code, the team must address those problems instead of moving forward.

As you can see, in this scenario the team battles time limits and manual interventions that don't need to exist because of automation. A build server will repeatedly do all this for you throughout the day, and without those human-caused delays.

But CI doesn’t just mean less time spent on manual tasks or the death of arbitrary deadlines, either. By automatically taking these steps many times a day, you’ll fix problems sooner and your results will become far more predictable. This, ultimately, helps you deploy through your pipeline with more confidence.

We’re taking deep dives into 2 CI platforms in the coming weeks, looking at how [Jenkins](https://www.jenkins.io/) or [GitHub Actions](https://github.com/features/actions) could help with your automation processes, plus how they complement Octopus.

## A little about Jenkins

[Jenkins](https://www.jenkins.io/) is the most popular CI platform on the market. Open-source and free to use, you can run Jenkins standalone on most operating systems to automate the building and testing of your code.

One of its biggest benefits, however, is its flexibility. For one, it’s a scalable platform, meaning you can expand its capabilities as your team or project needs grow. And thanks in no small part to its huge community, [there over 1800 plugins](https://plugins.jenkins.io/), making it easy to integrate with countless industry tools.
 
That means it’s not only flexible enough to cover your CI needs, but you can also tailor it for other automation purposes too.

## A little about GitHub Actions

[GitHub Actions](https://github.com/features/actions) is one of the newer CI platforms. It replaces the need for a separate build server by using repository events to trigger automation workflows on virtual ‘runners’. The good news for those using GitHub as their code repository is that you already have access - GitHub Actions is included in your existing repos.

Where’s the catch? While public repositories can use GitHub Actions for free, it’s pay-as-you-go for everyone else, billed by the minutes workflows take to run. All users get free monthly minutes, though, and you’re only charged if you exceed the number allowed by your plan.

Like Jenkins, [GitHub also has an actions marketplace](https://github.com/marketplace) brimming with community-created apps and workflows to help with CI and more.


## What's next?

You can look forward to more build server and CI content from us in the coming weeks, including guides for Jenkins and GitHub Actions, bespoke tools and more.

In the meantime, why not check out some of our older blogs on build servers, CI, and some of the ideas we’ll approach in this new series:

- [Continuous Integration vs. Continuous Deployment](https://octopus.com/blog/difference-between-ci-and-cd)
- [Octopus vs. build servers](https://octopus.com/blog/octopus-vs-build-server)
- [Integration 101: Octopus and build servers](https://octopus.com/blog/octopus-build-server-integration-101)
- [Java CI/CD: Octopus, Jenkins, Java, Kubernetes, and the DevOps lifecycle (series)](https://octopus.com/blog/java-ci-cd-co/)
- [The ten pillars of pragmatic deployments](https://octopus.com/blog/ten-pillars-of-pragmatic-deployments)

Happy deployments! 
