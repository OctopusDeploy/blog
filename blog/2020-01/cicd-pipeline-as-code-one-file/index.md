---
title: Should your CI/CD process be implemented in a single Pipeline as Code file?
description: As more teams move to Pipeline as Code, there are some important questions to ask when deciding to merge a CI/CD process into a single PaC file.
author: matthew.casperson@octopus.com
visibility: public
published: 2020-01-06
metaImage: pipeline-as-code.png
bannerImage: pipeline-as-code.png
bannerImageAlt: CI/CD Pipeline as Code
tags:
 - DevOps
---

![CI/CD Pipeline as Code](pipeline-as-code.png)

The concept of Pipeline as Code (PaC) was pioneered by build tools (i.e. Jenkins pipelines, Azure pipelines, GitLab CI/CD pipelines etc.) as a way of collocating the code being developed with the scripting required to build and test the code. This process immediately appealed to developers, and for this use case, PaC is ideal because the PaC logic and infrastructure it creates are very closely aligned with the code that it’s collocated with.

## Pipeline as Code for building and testing

The Pipeline as Code file shares the same lifetime as the code being committed. Like a unit test, PaC logic is designed to build and test the code it was committed with. And just as you wouldn’t use the code in unit tests from previous commits to validate the current state of the codebase, you likewise wouldn’t use a previous version of the PaC file to build and test the current codebase.

If the PaC logic creates any test infrastructure, that infrastructure would also be short-lived, likely for no longer than an hour or so. Tests that take more than an hour will frustrate developers and are generally not considered good practice.

Because the PaC logic is limited to building and testing code, the PaC process is owned and managed by the developers just as they own and manage the rest of the codebase.

The PaC workflow is a natural fit when it’s used as an extension of the build and test cycle. The same people are responsible for PaC code as the rest of the codebase, the PaC file lives and dies with each commit, and infrastructure created by the PaC logic is short-lived.

## Challenges extending a Pipeline as Code file for deployments

Naturally, there’s an inclination to extend Pipeline as Code beyond building and testing into deployments. At first glance, this appears to be the inevitable evolution of PaC, but there are good reasons not to have one pipeline for your entire CI/CD workflow.

While the process of building code and validating it through automated testing is measured in hours at most, the process of deploying a release through to production is exponentially longer; it’s not unheard of for release cycles to be measured in months.

As a consequence, the infrastructure involved in a deployment will also have an exponentially longer lifetime than the temporary infrastructure used for automated testing.

Deployments are also the responsibility of many people beyond the development team; any given release can be subject to the processes of product owners, QA, security teams, technical writers, and release managers.

Security also becomes a concern as different groups are ultimately responsible for different stages of the deployment process.

When a single PaC file is extended from building and testing into deployment, it moves from being the domain of developers looking to solve the very specific and time limited problem of compiling, validating, and packaging code to being subject to the many and varied requirements of multiple teams over a significantly extended timeframe. Modern software delivery practices encourage us to [distinguish releases from deployments](https://octopus.com/devops/continuous-delivery/deployments-vs-releases/). Separating the concepts relieves tension between development and operations.

If your deployment process is not completely automated, then by definition it requires human input, and any process that involves multiple teams over a period of days, weeks, or months will inevitably create a wide range of decision points, conflicting goals, and uncertainties around the state of the system. The developer-centric tooling that pioneered the PaC concept is often not well suited to dealing with the very human requirements of managing a long-running deployment workflow, forcing teams to try to represent these long-running and manual processes in a PaC file that was designed to support short-lived and disposable interactions.

## Conclusion

As tempting as it is to represent an entire CI/CD workflow with a single PaC implementation, anyone attempting to do so must first consider if these two processes are compatible enough from a business point of view to be merged, and if the tooling that hosts the PaC adequately supports the nonfunctional requirements of the deployment process.

Many teams will find these two processes have fundamentally different timelines, responsible parties, reporting requirements, and security restrictions. Even if the CI and CD processes are ultimately defined in code, they may be easier to manage as separate entities that can be edited, deployed, and secured with processes more aligned to their audience.
