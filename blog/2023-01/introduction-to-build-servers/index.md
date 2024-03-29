---
title: An introduction to build servers and Continuous Integration
description: This post opens our series about Continuous Integration and build servers, with an overview on the benefits of build servers, particularly Jenkins and GitHub Actions.
author: andrew.corrigan@octopus.com
visibility: public
published: 2023-01-30-1400
metaImage: blogimage-buildservers.png
bannerImage: blogimage-buildservers.png
bannerImageAlt: 2 people writing code on laptops with arrows to a build server, then an arrow to the octopus logo in a cloud, and a rocket launching
isFeatured: false
tags:
  - DevOps
  - CI Series
  - Continuous Integration
  - Jenkins
  - GitHub Actions
---

When you're developing and deploying software, one of the first things to figure out is how to take your code and deploy your working application to a production environment where people can interact with your software.

Most development teams understand the importance of version control to coordinate code commits, and build servers to compile and package their software, but Continuous Integration (CI) is a big topic. 

In our blog series on [Continuous Integration](https://octopus.com/blog/tag/CI%20Series), we go into detail about Continuous Integration and how 2 of the most popular build servers, [Jenkins](https://www.jenkins.io/) and [GitHub Actions](https://github.com/features/actions), can help with your CI processes.

## Why build servers are important

Build servers have 3 main purposes:

- Compiling committed code from your repository many times a day
- Running automatic tests to validate code
- Creating deployable packages and handing off to a deployment tool, like Octopus Deploy

Without a build server you're slowed down by complicated, manual processes and the needless time constraints they introduce. For example, without a build server:

- Your team will likely need to commit code before a daily deadline or during change windows
- After that deadline passes, no one can commit again until someone manually creates and tests a build
- If there are problems with the code, the deadlines and manual processes further delay the fixes

Without a build server, the team battles unnecessary hurdles that automation removes. A build server will repeat these tasks for you throughout the day, and without those human-caused delays.

But CI doesn’t just mean less time spent on manual tasks or the death of arbitrary deadlines, either. By automatically taking these steps many times a day, you fix problems sooner and your results become more predictable. Build servers ultimately help you deploy through your pipeline with more confidence.

## What is Jenkins?

[Jenkins](https://www.jenkins.io/) is the most popular CI platform on the market. Open-source and free to use, you can run Jenkins standalone on most operating systems to automate the building and testing of your code.

One of Jenkins biggest benefits is its flexibility. It's a scalable platform, meaning you can expand its capabilities as your team or project needs grow. And thanks to its huge community, [there are over 1800 plugins](https://plugins.jenkins.io/), making it easy to integrate with countless industry tools.
 
That means Jenkins is flexible enough to cover your CI needs and you can tailor it for other automation purposes too.

## What is GitHub Actions?

[GitHub Actions](https://github.com/features/actions) is one of the newer CI platforms. It removes the need for a separate build server by using repository events to trigger automation workflows on virtual ‘runners’. 

If you're using GitHub as your code repository, the good news is that you already have access - GitHub Actions is included in your existing repos.

Where’s the catch? While public repositories can use GitHub Actions for free, it’s pay-as-you-go for everyone else, billed per minute for the time workflows take to run. All users get free monthly minutes, though, and you’re only charged if you exceed the number allowed by your plan.

Like Jenkins, [GitHub also has an actions marketplace](https://github.com/marketplace) brimming with community-created apps and workflows to help with Continuous Integration and more. 

As an official GitHub technology partner, Octopus Deploy has a range of [verified Octopus actions in the GitHub Marketplace](https://github.com/marketplace?query=octopus&type=actions&verification=verified_creator) to help with your deployments. 

You can also learn more about [building with GitHub and deploying with Octopus](https://octopus.com/github).

## What's next?

In our [CI series](https://octopus.com/blog/tag/CI%20Series), we share guides for Jenkins and GitHub Actions, and more.

If you're not already using Octopus Deploy, you can [sign up for a free trial](https://octopus.com/start).

You can also check out some of other posts on build servers and CI:

- [Octopus vs. build servers](https://octopus.com/blog/octopus-vs-build-server)
- [Integration 101: Octopus and build servers](https://octopus.com/blog/octopus-build-server-integration-101)
- [Java CI/CD: Octopus, Jenkins, Java, Kubernetes, and the DevOps lifecycle (series)](https://octopus.com/blog/java-ci-cd-co/)
- [The ten pillars of pragmatic deployments](https://octopus.com/blog/ten-pillars-of-pragmatic-deployments)

Happy deployments!