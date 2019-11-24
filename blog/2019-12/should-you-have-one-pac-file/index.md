---
title: Should your CI/CD process be implemented in a single Pipeline as Code file?
description: As more teams move to Pipelines as Code, there are some important questions to ask when deciding to merge a CI/CD process into a single PaC file.
author: matthew.casperson@octopus.com
visibility: private
published: 2020-01-01
metaImage:
bannerImage:
tags:
 - Octopus
---

The concept of pipelines as code (PaC) was pioneered by build tools as a way of collocating the code being developed with the scripting required to build and test it. This process immediately appealed to developers, and for this use case PaC is ideal because the PaC logic and infrastructure it creates are very closely aligned with code that it is collocated with.

For example, the PaC file shares the same lifetime as the code. Like a unit test, PaC logic is designed to build and test the code it was commited with. And just as you would not use the code in unit tests from previous commits to validate the current state of the codebase, you would likewise not use a previous version of the PaC file to build and test the current codebase.

If the PaC logic creates any test infrastructure, that infrastructure would also be short lived, likely for no longer than an hour or so. Tests that take more than an hour will frustrate developers and are generally not considered good practice.

Because the PaC logic is limited to building and testing code, the PaC process is owned and managed by the developers just as they own and manage the rest of the code base.

The PaC workflow it is a natural fit when it is used as an extension of the build and test cycle. The same people are responsible for PaC code as the rest of the code base, the PaC file lives and dies with each commit, and infrastructure created by the PaC logic is short lived.

Naturally the inclination is to extend PaC beyond building and testing into deployments. At first glance this appears to be the inevitable evolution of PaC, but there are good reasons not to have one pipeline for your entire CI/CD workflow.

While the process of building code and validating it through automated testing is measured in hours at most, the process of deploying a release through to production is exponentially longer, as it is not unheard of for release cycles to be measured in months.

As a consequence, the infrastructure involved in a deployment will also have an exponentially longer lifetime than the temporary infrastructure used for automated testing.

Deployments are also the responsibility of many people beyond the development team; any given release can be subject to the processes of product owners, QA, security teams, technical writers and release managers.

And security becomes a concern as different groups are ultimately responsible for different stages of the deployment process.

When a single PaC file is extended from building and testing into deployment, it moves from being the domain of developers looking to solve the very specific and time limited problem of compiling, validating and packaging code to being subject to the many and varied requirements of multiple teams over a significantly extended timeframe.

If your deployment process is not completely automated, then by definition it requires human input, and any process that involves multiple teams over a period of days, weeks or months will inevitably create a wide range of decision points, conflicting goals and uncertainties around the state of the system. The developer centric tooling that pioneered the PaC concept is often not well suited to dealing with the very human requirements of managing a long running deployment workflow, forcing teams to represent long running and manual processes in a PaC file that was designed to support short lived and disposable interactions.

As tempting as it is to represent an entire CI/CD workflow with a single PaC implementation, anyone attempting to do must first consider if these two processes are compatible enough from a business point of view to be merged, and if the tooling that hosts the PaC adequately supports the nonfunctional requirements of the deployment process.

Many teams will find that these two processes have fundamentally different timelines, responsible parties, reporting requirements and security restrictions. Even if the CI and CD processes are ultimately defined in code, they may be easier to manage as separate entities that can be edited, deployed and secured with processes more aligned to their audience.
