---
title: Automatic rollbacks are a last resort
description: Find out why automatic rollbacks are rarely the right answer to deployment challenges.
author: steve.fenton@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 
isFeatured: false
tags: 
  - tag
---

People often say they want the ability to roll back software automatically when something goes wrong. This sounds simple enough, but there aren't many scenarios where an automatic rollback is desirable.

Many causes of deployment failures will also prevent a rollback. For example, if a credential has expired preventing part of the deployment, this will also fail during the rollback process. There may also be decisions to be made based on what operations succeeded before the deployment stopped, such as changes to database state.

There are three insights from modern software delivery that all contribute to the fall from grace of automatic rollbacks:

- We have a better understanding of resilience in our sociotechnical systems  
- Continuous Delivery makes it easier to roll forward  
- Progressive delivery provides better mechanisms

## The role of resilience

One of the crucial ideas from the Lean movement is that automation is improved if there are mechanisms to halt operation and ask for human intervention, instead of continuing automatic operation. This was termed *autonomation*, or automation with a human touch.

Although automatic rollbacks sound like something that would make your software more reliable, this is a situation where resilience is needed. Resilience is something humans excel in.

To put it another way, you can plan-in a certain level of robustness to your deployment process. For example, you could test credentials before starting the deployment. Each time you discover a new kind of failure you can learn from it and improve how it's handled. For all the things you haven't thought of, resilience is required.

Human-driven resilience provides the crucial knowledge needed to improve robustness. Organizations who flag the problem to a human end up with an appropriate resolution to the issue at hand, and knowledge of how to make the system more robust. Automatically rolling back removes this opportunity to learn as conditions for observing the problem may have been lost.

## Continuous Delivery: Fewer bugs and faster recovery

If rollbacks are a priority in your organization, it's likely a signal that you're missing crucial technical practices. Working in small batches is made possible by frequently committing changes to the main branch so they can be automatically validated to increase confidence that the new software version is deployable.

The longer you practice Continuous Delivery, the more robust your deployment pipeline becomes. Additionally, if something does slip through you can quickly roll forward to a new software version that has a fix (and some new tests that stop it from happening again). This is very different from using an expedited process for an urgent fix, Continuous Delivery lets you use the standard process for all changes, because it's already swift.

Most bugs you encounter within a Continuous Delivery process aren't critical. There may be an edge case you missed, some scenario that you might not decide to support, or cosmetic issues. Normal feedback cycles can handle these. When a change is more urgent, you can still resolve it by deploying a new software version.

Continuous Delivery progresses the same build artifact through each environment, it includes validation steps to make sure the software works to the extent you have defined tests, and the deployment process itself is being tested at least as often as the software, because you're using the same automated process to deploy to pre-production environments.

This means the kind of problems you encounter will be transient issues or unexpected events. Let's get those to an expert.

## Progressive delivery provides better strategies

Progressive delivery techniques include deployment strategies, such as blue/green and canary deployments, and feature flags.

Deployment strategies include mechanisms to verify the software with blue/green deployments providing a failback option, and canary deployments allowing you to pause the rollout to limit the impact of an issue. Feature flags take this a step further by providing a runtime mechanism to switch features on and off, or swap feature implementations.

## The last resort

Rollbacks don't always return you to a previous system state. They can return you to a state you've never tested or operated before. The software version may have been tested, but not with your current database or other dependencies. What other software has moved forward that may no longer be compatible with the previous version?

Redeploying a previous software version should be a last resort and it ought to have human supervision as there may be many other knock-on effects of moving in reverse. You may fix a bug only to re-open a security vulnerability, or revert to a software version that isn't compatible with the current database schema or some crucial API.

When you use modern software engineering practices like Continuous Delivery, modern deployment strategies, and feature flags, the appeal of automatic rollbacks is greatly diminished. Bringing human resilience to play results in the creation of increasingly robust systems for testing, deployment, and operation of software.

You can read more about this in the white paper, [Modern deployment and rollback strategies](https://octopus.com/whitepapers/modern-deployment-and-rollback-strategies) by Bob Walker. It provides advice on the architectural patterns that support better deployments, the trade-offs of different approaches, and how to select the right approach.

Happy deployments!
