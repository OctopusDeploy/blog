---
title: Achieving Continuous Delivery with Trunk-Based Development, Progressive Delivery, and Feature Toggles (TPF)
description: The best way to achieve the goals of Continuous Delivery is to implement Trunk-Based Development, Progressive Delivery, and Feature Toggles.
author: bob.walker@octopus.com
visibility: public
published: 2025-12-31-1400
metaImage: blog-generic-statistics-metrics-v2-2025-750x400-x2.jpg
bannerImage: blog-generic-statistics-metrics-v2-2025-750x400-x2.jpg
bannerImageAlt: Person standing in front of floating rectangles that contain different metrics.
isFeatured: false
tags: 
  - TPF
  - Continuous Delivery
---

Ten years ago, I was an Octopus Deploy customer.  After implementing Octopus Deploy (and Redgate for DB changes), my application went from quarterly deployments taking two (or more) hours to taking 15 minutes and deploying every ten(ish) days.  Even better, deployment failures and emergency fixes went from a virtual guarantee to never happening.  Our deployments were consistent and reliable.  It was a considerable accomplishment involving many people.  

But, we hadn't improved on how we _deliver new functionality_ to our users.  That was a problem, as new features and functionality carry tremendous risk.  There could be critical bugs, unknown use cases, performance, or UI issues with unexpected configurations.  In addition, many features fail to meet user expectations.  In the [Phoenix Project](https://www.amazon.com/Phoenix-Project-DevOps-Helping-Business/dp/0988262592), one main character states that only 10% of new features are helpful to users.

The problem I faced ten years ago was that we were still making feature changes like we did when we had monthly or quarterly releases.  

- Despite working on an internal application, we could not get direct feedback from users after hands-on usage. We often demoed new functionality to our Business Owner from our local machines to see if we were on the right track.
- The features were delivered in a nearly complete state. The feature branches lived for several weeks, resulting in merge hell.
- Main wasn't always ready to deploy.  After merging a feature into the main, finding and fixing all the bugs might take another week or two.
- When a critical bug needed to be fixed, we'd implement a simplistic configuration to turn on the feature for dev and QA but off for staging and production.
- We'd "turn on" the feature during a production deployment, but it'd be on for everyone. Users could access the new feature while we were verifying, resulting in bug reports or panicked emails (often because a third-party system hadn't finished updating).
- Over a half-dozen programming pairs all working in the same code base, causing more and more merge conflicts.
- Merge "freezes" preventing new features from being merged after deploying a new feature to allow a few days for hot fixes and performance improvements.

[Continuous Delivery](https://continuousdelivery.com) solves that by getting changes of all types—including new features, configuration changes, bug fixes and experiments—into production, or into the hands of users, _safely and quickly_ in a _sustainable_ way.  Production deployments should be predictable, routine affairs, that can be performed on demand.  That is achieved by ensuring the code is _always_ in a deployable state.  

To achieve Continuous Delivery, we needed to adopt TPF, or [trunk based development](https://trunkbaseddevelopment.com/), [Progressive delivery (canary, blue/green, or staging)](https://octopus.com/blog/common-deployment-patterns-and-how-to-set-them-up-in-octopus), and [Feature toggles](https://martinfowler.com/articles/feature-toggles.html). 

## Trunk based development

[Trunk based development](https://trunkbaseddevelopment.com/) promotes frequent check-ins to the main or default branch.  

> A source-control branching model, where developers collaborate on code in a single branch called "trunk", resist any pressure to create other long-lived development branches by employing documented techniques. 

The goals of trunk based development are:

- Avoiding merge hell: How many times have you worked on a long-lived feature branch, and merging to main took hours or days?
- Small batch sizes: Bigger batch sizes are much riskier to deploy than smaller ones.  A bigger batch size means more code has changed, meaning more can go wrong.
- Minimize [yak shaving](https://en.wiktionary.org/wiki/yak_shaving): Complex branching strategies, such as [GitFlow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow) are complex.  That complexity results in a lot of yak shaving to handle merging between branches, releasing, and other ceremonies.  

My personal preference is trunk based development "at scale."  

![Trunk based development diagram](https://trunkbaseddevelopment.com/trunk1c.png)

Source: https://trunkbaseddevelopment.

Trunk based development at scale works nicely with [GitHub flow](https://docs.github.com/en/get-started/using-github/github-flow), where changes are made in short-lived branches and merged via pull requests.

It requires developers to adopt two key concepts.

1. Short-lived feature branches - no more than a couple of days.  Ideally, less than a day.
1. Incremental changes - no more delivering a feature as a "big bang."  

That does not mean merging unfinished code into main after a few hours or a couple of days. Many features can take weeks or months to finish. Developers and product managers are responsible for creating small units of work that incrementally add functionality (and tests) until the feature is "done."  Each incremental change can be deployed to production. The feature should be hidden behind a feature toggle until it is ready for users.

## Progressive delivery

Trunk based deployments encourage frequent production deployments. However, you cannot deploy to production daily if each deployment has downtime. Any downtime might breach SLAs, frustrate customers, or prevent employees from doing their work. In addition, automation isn't perfect; a deployment could still fail. A progressive delivery, canary, blue/green, or staging is required to prevent deployment failures and zero-downtime deployments.

Every progressive delivery strategy follows the same core principle.  Deploy a new version to a "staging" location in production.  Verify the new version while all the users remain on the old version.  Once verification is complete, route users to the new version.  The difference between canary, blue/green, and staging is the percentage of traffic routed.

![Canary Deployment Diagram](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2021/07/14/ecs_canary_2.gif)

Source: https://aws.amazon.com/blogs/containers/create-a-pipeline-with-canary-deployments-for-amazon-ecs-using-aws-app-mesh/

Deployments can fail for a number of reasons.
- A database migration fails.
- Infrastructure cannot be created or updated properly.
- The new application version cannot be deployed due to environmental problems.
- The new application version won't start after deployments.
- The new application version crashes minutes after the deployment.

The advantage of progressive delivery is that every user remains on the old version while those failures are happening. Users don't have to be routed to the new version as soon as verification is completed. For example, you might deploy at 2 pm and wait to route traffic at 9 pm when there are fewer users.  

## Feature Toggles

Many people believe feature toggles are a simple binary.  It's either on or off for everyone.  Turn it on in testing environments, but leave it off for production.  Feature toggles must follow the specification provided by [Openfeature](https://openfeature.dev).  Instead of an application-level binary, multiple user segments are created.  The feature toggle is then turned on for each segment.  

![Feature Toggle Segments](FeatureToggleSegments.png)

The segments can be whatever you want them to be:

- Staff or internal users
- Early adopters or beta users
- Timezone
- State or Province
- Country
- Specific entitlements

The advantages to this approach are numerous:

1. Internal or staff users can [dogfood](https://en.wikipedia.org/wiki/Eating_your_own_dog_food) new functionality in production before anyone else.
2. Provide early access to a subset of alpha users for early feedback.
3. Beta-test new functionality with a large group of users before it goes live for everyone.
4. Being able to pivot and change functionality based on user feedback.
5. Rolling back involves turning off a feature toggle, not redeploying a previous version.
6. Turning off the toggle is often low-risk, as it has already been off for weeks or months in production.

## Rollbacks become a thing of the past

The number one driver of rolling back is critical bugs in new functionality.  By adopting TPF, you are preventing that from happening by: 

- Incrementally adding functionality instead of merging everything as a "big bang."
- Providing the capability _from the start_ of the new feature to turn it off.
- Frequently deploying with that functionality turned off.

That is not to say every feature will be bug-free, far from it.  You'll always encounter unknown use cases, random configurations you weren't expecting that cause your feature to fail, or substandard performance.

The big difference is that with feature toggles, you can turn off that functionality for specific users. Before TPF, you'd roll back to a previous version and turn it off _for all users_. Imagine being able to say to a user, "We've identified the root cause. We'll push out a fix overnight. In the meantime, we've turned it off for you to prevent any issues."

The deployment pipeline should be treated like an assembly line.  Always moving forward.  Never backward.  

## TPF and Octopus Cloud

In the early days of Octopus Deploy, we implemented canary deployments using release rings.  

- We'd deploy to our staff instances and wait a day to see if anything blew up.
- After a day, we'd deploy to the Octopus Insiders and get their feedback.
- Assuming none of the Octopus Insiders found problems, we'd deploy to our early adopters after seven days.  
- We'd wait seven says, and if the early adopters didn't report anything, we'd deploy to the stable customers.
- Then we'd deploy to any laggards (very rarely did we have any).

![Octopus Cloud v1 Canary](ReleaseRingsOnOctopusCloud.png)

We thought that canary deployments would be a panacea.  But in truth, it had many limitations.

1. No guarantee all code paths are executed by a subset of users.  Unless you are a FAANG-sized company, a subset of traffic is unlikely to execute every code path in a short period of time.  
2. Long deployment durations.  The natural reaction to the code path limitation is to extend the canary deployment duration to hours or even days.
3. Indiscriminate traffic routing.  Targeting a specific set of users via a load balancer is nearly impossible.  In the best case, you can use HTTP headers, but that is often too broad.
4. Disjointed user experience. When canary deployments take days or even weeks, and traffic routing is indiscriminate, you cannot rely on "sticky sessions" to keep a user on a particular version.
5. User expectations.  I have a laptop and an iPad.  When I go to a website, I expect the same experience when using my laptop or iPad (even when used within minutes of one another).

Even with a 15-day canary cycle, critical bugs still got to the stable ring on Octopus Cloud! The 15-day cycle was the best case, as we might find a show-stopping bug. At one point in 2022, customers in the stable ring didn't get updates for almost 50 days!

Our engineering leaders (managers and senior engineers), realized we needed to change.  We adopted TPF.

- Trunk based development with short lived branches, no more than a few days.
- Feature toggles, which could be turned on or off for a specific customer instance.
- Shorten release rings, and deploy to our main instance that deploys Octopus Deploy (we use Octopus Deploy to deploy Octopus Deploy).

![Octopus Cloud v2 Canary](ReleaseRingsOnOctopusCloud_v2.png)

There have been numerous benefits to this approach.

- We have four concurrent early access programs for features on our [roadmap](https://roadmap.octopus.com/tabs/2-planned).  Each program has a different list of customers.
- We can incrementally add new functionality to features and gather feedback. For example, I have early access to process templates, and every other day, I see the feature taking shape. This gives us the opportunity to get feedback from our customers (not just from me) and adjust before releasing it to everyone.
- We can eat our own dog food and find the rough spots.  For example, internally we were using the updated Octopus Deploy UI released last year for months.  
- Rolling out new functionality is an update to a feature toggle.  Even better, if a customer reports an issue, we can turn it off to unblock them.  Meanwhile, everyone else can use the new functionality.  

## Conclusion

TPF, trunk based development, progressive delivery, and feature toggles are a tripod.  All three are needed to achieve true Continuous Delivery; getting changes of all types—including new features, configuration changes, bug fixes and experiments—into production, or into the hands of users, _safely and quickly_ in a _sustainable_ way.  Without trunk-based development, you cannot iteratively make changes to a feature to gather feedback faster.  Without progressive delivery, you increase the risk of a deployment failure or prolonged downtime.  And without feature toggles, you cannot get feedback for the feature from a subset of users, or if something wrong happens, unable to turn off the new feature.

We have seen two revolutionary jumps happen with our customers.  The first one is from two-hour deployments to 15-minute deployments.  The second one is from 15-minute deployments to zero downtime.  The first revolution is accomplished by implementing Octopus Deploy, having a single deployment process for all environments, automating the deployment for all components, and leveraging runbooks for day 0 and day 2 operations.  The second revolution is accomplished by implementing TPF.  In 2025, expect more functionality in Octopus Deploy to support this.  [Feature Toggles](https://roadmap.octopus.com/c/121-feature-toggles) and [Ephemeral Environments](https://roadmap.octopus.com/c/31-ephemeral-environments) are actively being worked on.  With easier [modeling of canary and blue/green deployments](https://roadmap.octopus.com/c/145-easy-blue-green-and-canary-modeling) is being shaped.

Until next time, Happy Deployments!
