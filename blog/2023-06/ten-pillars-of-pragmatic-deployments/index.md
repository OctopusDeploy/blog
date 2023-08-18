---
title: The ten pillars of pragmatic deployments
description:  The ten pillars of pragmatic deployments that shape how we develop the features and philosophy of Octopus Deploy.
author: matthew.casperson@octopus.com
visibility: public
published: 2023-06-07-1400
metaImage: blogimage-10-pillars-2023x2.png
bannerImage: blogimage-10-pillars-2023x2.png
bannerImageAlt: Building with ten pillars
isFeatured: true
tags:
 - DevOps
 - Deployment Patterns
---

As you might imagine, at Octopus, we spend a great deal of time thinking about deployments. Octopus was born to manage real-world deployment processes. We constantly shape the features through conversations with customers, support requests, and our own internal deployment requirements and discussions about what we think good deployments look like.

We have forged all this deployment knowledge into the 10 pillars of *pragmatic* deployments.

We use the phrase "pragmatic" quite deliberately. This list is not a scorecard, a pyramid, a checklist, or a fixed list of requirements. One of the goals of developing the 10 pillars is to find gaps in our processes, shape our product's features and philosophy, and continue the ongoing discussion about what it means to deploy software.

With that in mind, pillar zero, the prerequisite for all other pillars, is *do what works for you*.

We hope you enjoy this discussion and look forward to shaping the future of pragmatic deployments with your feedback.

## The ten pillars

- [Repeatable deployments](#repeatable-deployments)
- [Verifiable deployments](#verifiable-deployments)
- [Seamless deployments](#seamless-deployments)
- [Recoverable deployments](#recoverable-deployments)
- [Visible deployments](#visible-deployments)
- [Measurable deployments](#measurable-deployments)
- [Auditable deployments](#auditable-deployments)
- [Standardized deployments](#standardized-deployments)
- [Maintainable deployments](#maintainable-deployments)
- [Coordinated deployments](#coordinated-deployments)


## Pillar 1. Repeatable deployments {#repeatable-deployments}

Software teams are deploying software to production more frequently than ever. They also make deployments to pre-production environments as part of their deployment pipeline.

To be confident when you change software at high velocity, you need a mix of methods such as test automation, exploratory testing, self-beta testing (drinking your own champagne), and other techniques to confirm the release-ability of your software.

These techniques are only helpful if what you deploy to production is the same as what you deployed to your other environments.

The pillar of repeatable deployments requires you to deploy the same thing in the same way each time you deploy an application version.

### General deployment concepts

To understand repeatable deployments, we need to fine-tune our definitions. We must be precise with some terms relating to deployment pipelines and the timing of different stages.

#### Deployment pipelines

A deployment pipeline starts when you commit code to a source code repository, and it follows the change all the way to the production environment.

Every activity needed to progress the change is part of your deployment pipeline. Code reviews, builds, testing, release management, sign-offs, and deployments are all included, whether manual or automated.

#### Continuous integration

The most common entry point to a deployment pipeline is *continuous integration* (CI). This is the process of committing code, compiling it, running tests, packaging a new application version, and publishing it.

Dave Farley and Jez Humble recommend changes are regularly integrated into the main branch in source control (hence the "continuous" in "continuous integration"). Many teams use the term more loosely to define their automated build process.

#### Continuous Delivery

While continuous integration is the automation of creating a deployable package each time the code changes, [Continuous Delivery](https://octopus.com/devops/continuous-delivery/) (CD) extends this through to deployment automation and monitoring.

You might still have some manual stages in your deployment pipeline, such as exploratory testing or an approval process. But you should automate all deployment steps as this helps you achieve repeatable deployments.

Using the same deployment process for all environments means it gets tested as often as the application.

#### Continuous Deployment

[Continuous Deployment](https://octopus.com/devops/continuous-delivery/what-is-continuous-deployment/) takes CI/CD a step further, removing all manual intervention to create a fully automated commit-to-consumer workflow. The deployment pipeline automatically rejects a bad application version and deploys all good versions to production without manual approvals.

#### CI/CD

The term CI/CD refers to the combination of continuous integration with either Continuous Delivery or Continuous Deployment.

Teams commonly deploy to the development environment without manual intervention but allow people to control when they deploy to subsequent environments. We have learned from most of our customers that this approach *works well for them*.

In the test environment, quality assurance (QA) staff validate changes, product owners review new functionality, security teams probe for vulnerabilities, etc. When everyone is happy that the changes meet their requirements, they can promote the application version to production.

The production environment is the final destination. It's where end users can use the application.

#### What is an environment?

Environments represent the boundaries between copies of individual applications or entire application stacks and their supporting infrastructure. 

Each environment should reasonably reflect the production environment. This ensures the application will behave consistently and avoids surprises when you go live.

You might deploy more frequently to earlier environments, trading stability for faster feedback. Your users expect production environments to be stable.

You progress deployments through environments to increase confidence that you can deliver a working solution to the end user.

We call the canonical set of environments development, test, and production. The table below describes the characteristics of these environments:

| Environment | Description | Deployment Frequency | Stability / Confidence |
|-|-|-|-|
| Development | Used by developers to test individual changes as they are implemented. | High | Low |
| Test | Used by developers, QA, and non-technical staff to validate that changes meet requirements. | Medium | Medium |
| Production | Accessed by end users to consume the publicly available instance of the applications. | Low | High |

Although you can have any number of environments with different names, we use this set of environments in the examples.

#### What is a deployment?

We've talked about deploying "applications" to environments, but to appreciate how you achieve repeatable deployments, we must be more specific about what we deploy.

A deployment should include a snapshot of:

1. The application version
2. The deployment process
3. The variables used to configure the application for an environment
4. Inline scripts that support the deployment and configuration of the application and infrastructure

Octopus combines these into a *release*, which captures the current deployment state. The release snapshot ensures you're using the same snapshot for the release, even if you change the process, variables, or scripts during environment progression.

Without a release snapshot, you could deploy a tested application version using an untested deployment process, resulting in a problem in your production environment.

## Pillar 2. Verifiable deployments {#verifiable-deployments}

The repeatable deployments pillar describes how promoting releases through environments increases confidence in the application version and deployment process.

The pillar of verifiable deployments describes the various techniques you can use to verify a deployment when it reaches a new environment.

### General testing concepts

Testing is a nebulous term with often ill-defined subcategories. We will not attempt to provide authoritative definitions of testing categories here. We aim to offer a high-level description of common testing practices and highlight those you can use when you deploy.

#### What don't we test during deployments?

We consider unit tests part of the build pipeline. They're closely tied to the code and must pass for the build server to publish the application package.

You might also run integration tests during the build process to verify components interact as expected. Sometimes, you might use a [test double](https://martinfowler.com/bliki/TestDouble.html) instead of a real component to improve reliability. Otherwise, you might spin up the dependencies as part of the test.

The tests run by the build server should result in a high-quality application package being published, with bad application versions prevented from progressing.

#### What can we test during deployment?

Tests that require a live application or application stack to be accessible are ideal candidates to run as part of a deployment process.

Smoke tests are quick tests designed to ensure you deploy applications and services correctly. Smoke tests implement the minimum interaction required to ensure services respond correctly. Some examples include:

- An HTTP request of a web application or service to check for a successful response.
- A database login to ensure the database is available.
- Checking that a directory has been populated with files.
- Querying the infrastructure layer to ensure the expected resources were created.

Integration tests can run as part of your build or deployment processes. Integration tests validate that multiple components are interacting as you expect. You may include test doubles with the deployment to stand in for dependencies, or the tests may verify two or more running component instances.

Examples include:

- Signing into a web application to verify that it can interact with an authentication provider.
- Querying an API for results from a database to ensure that the database is accessible via a service.

End-to-end tests provide an automated way of interacting with a system like a user would. These can be long-running tests following paths through the application that require most or all application stack components to work correctly. Examples include:

- Automating the interaction with an online store to browse a catalog, view an item, add it to a cart, complete the checkout, and review the account order history.
- Completing a series of API calls to a weather service to find a city's latitude and longitude, getting the current weather for the returned location, and returning the forecast for the rest of the week.

Chaos testing involves deliberately removing or interfering with the components that make up an application to validate that the system is resilient enough to withstand such failures. You can combine chaos testing with other tests to verify the stability of a degraded system.

Usability and acceptance testing often involve a human using the application to verify that it meets the requirements. The requirements can be subjective, for example, determining if the application is visually appealing. Or the testers may be non-technical and don't have the option of automating tests. The manual and subjective nature of these tests makes them difficult, if not impossible, to automate, meaning a working copy of the application or application stack must be deployed and made accessible to testers.

## Pillar 3. Seamless deployments {#seamless-deployments}

When you deploy your application, you need to switch users from the old application version to the new one. The pillar of seamless deployments discusses approaches to reducing the user impact when you update the application.

One of the easiest ways to deploy an application without impacting users is to use planned maintenance windows. You can schedule these to occur outside of the users' normal working hours to minimize disruption to their work.

This approach is less practical if you have users in many time zones, need to minimize downtime, or want to deploy more often.

In these cases, you can use some common deployment strategies to deploy new application versions seamlessly.

### Seamless database deployments

No discussion on seamless deployments can begin without first addressing the issue of database updates.

A fundamental aspect of most seamless deployment strategies involves running two versions of your application side by side, if only for a short time. If two versions of the application access the same database, all updates to the database schema and data must be compatible with both versions. We refer to this as backward and forward compatibility.

However, backward and forward compatibility is not trivial to implement. In the presentation [Update your Database Schema with Zero Downtime Migrations](https://www.youtube.com/watch?v=3mj6Ni7sRN4) (based on chapter 3 of the book [Migrating to Microservice Databases](https://developers.redhat.com/books/migrating-microservice-databases-relational-monolith-distributed-data)), Edison Yanaga walks through the process of renaming a single column in a database. It involves six incremental updates to the database and application code, and you must deploy each in sequence.

Seamless deployments involving a database require careful planning, many small steps to roll out the changes, and tight coordination between the database and application code.

You can find more database techniques in [Refactoring Databases](https://www.thoughtworks.com/en-us/insights/books/refactoring-databases) by Scott W. Ambler and Pramod Saalage.

### Deployment strategies

There are several ways to manage a cutover between an existing and a new application version. The right deployment strategy can help you achieve seamless deployments.

#### Recreate

The recreate strategy doesn't provide a seamless deployment. It's included here as the default option for most deployment processes. This strategy involves either:

- Removing the existing deployment before deploying the new version.
- Deploying the new version over the top of the current one.

Both options result in downtime between the existing version being stopped or removed and the new version starting. However, because you don't run the current and new versions concurrently, your database upgrade won't need to satisfy backward and forward compatibility requirements.

#### Rolling updates

The rolling update strategy involves incrementally updating instances of the current deployment with the new deployment. This strategy ensures at least one application instance is available at all times during the rollout.

Your load balancer will send users to instances with its usual balancing pattern, so as the rollout progresses, more users will be moving onto the new application version. You can also take each instance out of balance before you update it, so requests aren't dropped during the deployment.

With rolling updates, your database will need to maintain backward and forward compatibility, as during the rollout it will receive connections from both the old and new application versions.

#### Canary deployments

The canary deployment strategy is similar to the rolling update strategy in that both incrementally expose more end users to the new deployment over time.

With canary deployments, you create a small group of users who get new versions before everyone else. This group may contain users who opt-in to get early access, represent a segment of your business (such as free-tier users), or be an algorithmic sample.

The first step of a canary deployment is to update the application instance used by your canary users. Once you have collected information from this group, you can decide whether to continue the rollout. This may be a human decision, or you might automate it based on monitoring data or log files.

Canary deployments allow you to halt the rollout and revert to the previous application version if you find a problem in the sample group.

#### Blue/green deployments

The blue/green strategy involves deploying the new version (the green version) alongside the current version (the blue version) without exposing the green version to any traffic. After you deploy and verify the green version, traffic is cutover from the blue to the green version. When the green version handles all traffic, you can remove the blue version.

Any database changes deployed by the green version must maintain backward and forward compatibility. Even when the green version is not serving traffic, the blue version gets exposed to database changes.

This deployment strategy requires additional infrastructure during the deployment process, but this can be created and destroyed automatically or used as a cold standby, depending on your needs and cost constraints.

#### Session-draining

You can use the session-draining strategy when applications maintain states tied to a particular application version.

This is similar to the blue/green strategy as it also means you deploy the new version alongside the current version, running both side by side. Unlike the blue/green strategy, session-draining will direct only new sessions to the new version while the existing one continues to serve traffic for existing sessions.

You need the same infrastructure and clean-up requirements for blue/green and session-draining strategies.

Any database changes must maintain backward and forward compatibility because the old and new application versions run side-by-side.

#### Feature flags

The feature flag strategy involves building functionality into a new application version and using a toggle or feature flag to control its visibility. This lets you control feature visibility without needing a deployment.

In practice, the deployment of a new application version with flaggable features will be performed with one of the strategies above, so the feature flag strategy is a complement to those other strategies.

#### Feature branch

The feature branch strategy allows developers to deploy an application version with changes they are working on, usually in a non-production environment, alongside the main deployment.

Maintaining database backward and forward compatibility may not be necessary with feature branch deployments. Because feature branches are for testing and are short-lived, it is acceptable that each feature branch deployment has access to its own test database.

## Pillar 4. Recoverable deployments {#recoverable-deployments}

Despite your best efforts with repeatable and verifiable deployments, you'll always need to handle production bugs. When this happens, being able to quickly and safely recover is crucial.

### Rolling back or forward

Recovering from an undesirable deployment means choosing whether to roll back to a previous good application version or roll forward to a new version that returns the environment to a desirable state.

Either solution works with stateless applications, but you must treat rollbacks with care when there's a database involved.

This is the [advice from the FlyWay project](https://flywaydb.org/documentation/command/undo#important-notes):

> While the idea of undo migrations is nice, unfortunately it sometimes breaks down in practice. As soon as you have destructive changes (drop, delete, truncate, …), you start getting into trouble. And even if you don't, you end up creating home-made alternatives for restoring backups, which need to be properly tested as well.

[RedGate offers this advice for database rollbacks](https://documentation.red-gate.com/sca/developing-databases/concepts/advanced-concepts/rollbacks/rollback-guidance):

> Rather than investing time and energy into rollback planning, an alternative is to follow an approach that keeps you moving forward.

The blog post [Pitfalls with SQL rollbacks and automated database deployments](https://octopus.com/blog/database-rollbacks-pitfalls) has this advice:

> More often than not, the effort to successfully rollback a deployment far exceeds the effort it would take to push a fix to production.

When deployments involve database changes, I recommend rolling forward to recover from an undesirable deployment.

### Rolling back

If you have achieved the pillar of *repeatable deployments* you can roll back by re-running a previous deployment. This is possible because the package versions, scripts, and variables are all captured by a repeatable deployment.

Rollbacks are also an explicit feature of several seamless deployment strategies:

- Canary deployments implement rollbacks by redirecting all traffic from the new deployment to the current deployment. 
- Blue/green deployments can roll back a deployment by cutting traffic back to the blue stack. 
- Session-draining deployments can redirect new sessions to the current deployment and optionally kill any sessions in the new deployment.

Rollbacks have the following benefits:

- You can fix a deployment issue, without writing code, by rolling back to a previous deployment.
- A rollback leaves the system in a known, verified state.
- You can measure the time to complete a rollback in non-production environments.

Rollbacks have the following disadvantages:

- Rollbacks are all-or-nothing operations. You can not roll back individual features, only entire deployments.
- You need to test rollbacks as part of the deployment process to ensure they work as expected, which increases the complexity and time of the deployment process.
- If a rollback fails, it is likely that you will need to resolve the issue by rolling forward.
- Database rollbacks require special consideration to ensure data is not lost.

### Rolling forward

Rolling forward is another way to describe performing a new deployment. In this case, the new deployment will only contain the fixes required to restore an environment.

Rolling forward has the following benefits:

- All deployment strategies, with or without a database, inherently support rolling forward.
- Teams gain experience in rolling forward with every deployment.
- You can choose the scope of a change or fix when rolling forward.
- Multiple deployments can be made in succession while rolling forward to resolve an undesirable deployment.

Rolling forward has the following disadvantages:

- Rolling forward typically requires a developer to implement a fix to include in the next deployment.
- You must have a short lead time for changes. Otherwise, you'll be tempted to skip environments to expedite the fix.
- The production environment will be left in an undesirable state for as long as it takes to develop and deploy the next version.

## Pillar 5. Visible deployments {#visible-deployments}

It can be challenging to track which application version you deployed to each environment. You shouldn't have to review the files on the disk or the structure and data in the database. This is like trying to work out what mix of colors produced a tin of paint.

Having a view of environments and application versions is crucial to understanding:

- What features you've provided to your customers
- What features are being tested
- What issues you've fixed
- The history of any changes

Listed below are the details required to gain complete visibility into the state of your deployments.

### Commit messages

Commit messages capture the intention of source code edits, describing the changes made and who made them. These messages are invaluable when trying to understand at a low level what changes made it into a particular version of a package.

### Issue tracking

Often source code commits are made to resolve an issue documented in a dedicated issue tracker. These issues provide a space for you to describe, discuss, and track bugs. A unique identifier references each issue.

Capturing the issue IDs that relate to changes in a package version and any deployment that includes that package version, provides insight into the issues resolved in any given deployment.

### Build logs

A typical CI/CD pipeline will have a build server that builds, tests, and packages an application. The log files for these builds contain a wealth of information, such as:

- Which tests passed
- Which tests were ignored
- Which dependencies were included
- What packages were created

You can quickly review these log files if you have a link to the build information from the deployment.

### Library dependencies

Almost every application deployed today combines custom code with third-party, often open-source, libraries. These external libraries provide useful features but can be a source of bugs or security vulnerabilities. Understanding all the application's dependencies is essential for security, auditing, and debugging.

### Deployment versions

A release version captures a snapshot of all of the above information, along with the deployment process, package versions, variable values, and scripts. You deploy this release version to each environment.

Displaying which release versions have been deployed to each environment provides a high-level view of the state of your deployments. With this information, anyone can see what's deployed where. By drilling into the details of a release, you can see the commit messages, issues, dependencies, and links to the CI builds.

## Pillar 6. Measurable deployments {#measurable-deployments}

The deployment pipeline's primary goal is getting your software into your customers' hands. To understand how well your deployment pipeline is performing, you need to measure the metrics that define success for you.

Having measurable deployments means defining useful metrics, reliably collecting them, and surfacing them in an easy-to-understand format.

Some metrics you can use to track deployments are:

- Deployment frequency: How frequently do you deploy to production?
- Lead time for changes: How long does it take for a code change to be deployed to production?
- Time to recover deployment: How long does it take to recover from a failed deployment?
- Deployment fail rate: What is the ratio of failed to successful deployments? 
- Change fail rate: What is the ratio between hotfix and regular deployments? 
- Deployment duration: How long does each deployment take?

## Pillar 7. Auditable deployments {#auditable-deployments}

If the visible deployments pillar is about surfacing the current state of your environments and the changes made as part of the release, then auditing is about tracking who has been involved in the deployment process over time.

Auditing allows teams to see a history of all deployment activity, like:

- Deployments to environments
- Changes to the deployment process
- Changes to environments
- Who approved a deployment
- The state of an environment at some point in the past

For audit events to be helpful, they must be searchable, filterable, and exportable to support reporting and analysis.

## Pillar 8. Standardized deployments {#standardized-deployments}

Just as repeatable deployments build confidence as you promote a release across environments, standardizing deployment processes across different projects allows teams to confidently use proven processes.

There are two main aspects to standardizing deployments.

The first aspect is defining the initial deployment process used by the various deployments. This base deployment process can be a shared template that allows you to customize specific settings. Or you can copy and paste the entire process to bootstrap new projects, allowing teams to customize the process to their needs.

The second aspect is defining who can edit a deployment process. By distinguishing between the ability to view, run, create, and edit a deployment process, teams can guarantee that only individuals responsible for creating or editing deployment processes can do so.

## Pillar 9. Maintainable deployments {#maintainable-deployments}

Getting your deployments to the production environment is just the beginning. Diagnosing issues, collecting logs, performing backups, restarting services, rotating keys, and testing connections are just some of the day-to-day operations tasks that keep your applications running and your customers happy.

While you could SSH or RDP into a server and start poking around, each change you make causes your environment configuration to drift, making it harder to implement repeatable deployments. It is also difficult to track changes, verify that they worked, and audit who changed what.

Maintenance tasks should be repeatable, verifiable, visible, measurable, auditable, standardized and coordinated - just like deployments. Maintenance tasks represent the business knowledge required to keep your deployments running and should meet the same standards as your deployment processes.

## Pillar 10. Coordinated deployments {#coordinated-deployments}

Deploying a package to an environment is just one small part of the deployment process. Often, you need to coordinate deployments with other business processes to ensure:

- The right people have given their approval
- Interested parties get notified of the success or failure of a deployment
- Deployments proceed in the correct order
_ Deployments can only occur during specific times
- High-priority deployments take precedence over low-priority ones
_ Deployments get scheduled to take place at a predetermined time
- External events can trigger deployments
- Deployments can trigger external events

A deployment process may be a single component in the broader ecosystem of business process management tools. The ability to orchestrate deployments from third-party platforms and report results back allows teams to manage complex deployments as part of a broader business process.

## Conclusion 

Those are the 10 pillars of pragmatic deployments that shape the [features](https://octopus.com/features) and philosophy of Octopus Deploy. I'm sure they'll continue evolving as new use cases emerge, but we believe they provide a solid foundation for building Octopus Deploy.

You might also like to read our ebook about the [10 pillars of pragmatic Kubernetes deployments](https://octopus.com/blog/10-pillars-kubernetes-deployments) with Octopus.

Happy deployments!
