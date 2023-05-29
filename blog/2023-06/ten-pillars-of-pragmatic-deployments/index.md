---
title: The ten pillars of pragmatic deployments
description:  The ten pillars of pragmatic deployments that shape how we develop the features and philosophy of Octopus Deploy.
author: matthew.casperson@octopus.com
visibility: public
published: 2021-02-22-1400
metaImage: blogimage-10-pillars-2023x2.png
bannerImage: blogimage-10-pillars-2023x2.png
bannerImageAlt: Building with ten pillars
isFeatured: true
tags:
 - DevOps
 - Deployment Patterns
---

![The ten pillars of pragmatic deployments](blogimage-the-ten-pillars-of-pragmatic-deployments-2021.png)

As you might imagine, at Octopus, we spend a great deal of time thinking about deployments. Octopus was born to manage real world deployment processes and continues to be shaped through conversations with customers, support requests, our own internal deployment requirements, and many other discussions about what we think good deployments look like.

This knowledge has been distilled into the 10 pillars of pragmatic deployments.

We use the phrase "pragmatic" quite deliberately. This list is not a score card, a pyramid, a checklist, or a fixed list of requirements. In many cases Octopus, both as a product and as a company, has not achieved these pillars. One of the primary goals of developing these pillars is to find the gaps in our own processes in order to shape the features and philosophy of our product, and to continue the ongoing discussion about what it means to deploy software.

With that in mind, pillar number zero, and the prerequisite for all other pillars in this list, is *do what works for you*.

We hope you enjoy this discussion, and look forward to shaping the future of pragmatic deployments with your feedback.

## The ten pillars

* [Repeatable deployments](#repeatable-deployments)
* [Verifiable deployments](#verifiable-deployments)
* [Seamless deployments](#seamless-deployments)
* [Recoverable deployments](#recoverable-deployments)
* [Visible deployments](#visible-deployments)
* [Measurable deployments](#measurable-deployments)
* [Auditable deployments](#auditable-deployments)
* [Standardized deployments](#standardizaed-deployments)
* [Maintainable deployments](#maintainable-deployments)
* [Coordinated deployments](#coordinated-deployments)


## Pillar 1. Repeatable deployments {#repeatable-deployments}

One of the primary reasons to progress a deployment through environments is to gain increasing confidence that you are providing a working solution to your end users. This confidence can be built through testing (both manual and automated), manual sign off, using your own software internally (drinking your own champagne), early releases to test users, or any number of other processes that allow issues to be identified before they impact your end users.

However, you only gain this confidence if what you are deploying to production is as close as possible to what you have been verifying in non-production.

By embracing repeatable deployments, you can be sure that what your end users use in production is what you have been testing, verifying, and gaining confidence in through your non-production environments.

### General deployment concepts

To understand repeatable deployments, we need to understand what a deployment is, and at what point in a typical build and deployment pipeline deployments take place.

#### Continuous Integration, Continuous Delivery and Continuous Deployment

The terms Continuous Integration and Continuous Delivery or Deployment (CI/CD) are frequently used to describe the progression from source code to publicly accessible application.

We consider Continuous Integration (CI) to be the process of compiling, testing, packaging, and publishing an application as source code is updated.

Continuous Delivery and Continuous Deployment (both abbreviated to CD) have subtly different meanings. We treat both terms as an automated series of steps that deliver an application to its destination. The distinction is whether those automated steps deliver the application directly to the end user with no manual human intervention or decision making.

Continuous deployments have no human intervention. You have achieved continuous deployments when a commit to your source code is compiled, tested, and packaged by your CI server, and then deployed, tested and exposed to end users. The success or failure of each stage of this process is automatic, resulting in a commit-to-consumer workflow.

Continuous delivery involves a human making the decision to progress a deployment to end users. This progression is typically represented as a promotion through a series of environments. A canonical example of environmental progression is to deploy applications to the non-production development and test environments, and finally to the production environment. 

A development environment may very well be configured with continuous deployments, where each commit successfully built by the CI server is automatically deployed with no human intervention. When the developers are happy that their changes are suitable for a wider audience, a deployment can be promoted to the test environment.

The test environment is where quality assurance (QA) staff validate changes, product owners ensure functionality meets their requirements, and security teams probe for vulnerabilities, etc. When everyone is happy that the changes meet their requirements, a deployment can be promoted to production.

The production environment is the final destination of a deployment, and this is where applications are exposed to end users.

We have learned from most of our customers that continuous delivery *works for them*. So while the majority of the pillars apply equally well to continuous delivery and continuous deployment, we'll approach them from a continuous delivery point of view.

#### What is an environment?

Environments represent the boundaries between copies of individual applications or entire application stacks combined with their supporting infrastructure. 

The configuration of all environments should be as similar as possible to ensure that an application will behave consistently regardless of which environment it exists in.

Environments typically represent a progression from an initial environment with a high frequency of deployments and low stability through to the final environment with a low frequency of deployments and high stability.

Deployments are progressed through environments to gain an increasing level of confidence that a working solution can be delivered to the end user.

The canonical set of environments are called development, test, and production. The table below describes the characteristics of these environments:

| Environment | Description | Deployment Frequency | Stability / Confidence |
|-|-|-|-|
| Development | Used by developers to test individual changes as they are implemented. | High | Low |
| Test | Used by developers, QA, and non-technical staff to validate changes meet requirements. | Medium | Medium |
| Production | Accessed by end users to consume the publicly available instance of the applications. | Low | High |

Although you are free to have any number of environments with any names, this set of environments will be used in the examples.

#### What is a deployment?

We've talked about deploying "applications" to environments, which is typically how we describe deployments. But to appreciate how repeatable deployments are achieved, we first need to be specific about what we actually deploy.

There are three things that can be deployed to an environment:

1. The compiled applications that are configured for a specific environment. These are referred to as packages.
2. The variables, usually with a small subset specific to individual environments, that define how the applications are configured.
3. Scripts and configuration files written inline (i.e., not saved as files in packages) to support or configure the application and its associated infrastructure in an environment.

The package versions, variable values, and scripts or configuration files are captured as a release.

The release is then deployed to an environment. In this way a consistent bundle of packages, variables, scripts and configuration files are promoted from one environment to the next. Only a small subset of environment specific settings vary from one environment to the next.

## Pillar 2. Verifiable deployments {#verifiable-deployments}

The repeatable deployments pillar describes how promoting releases through environments provides an increasing level of confidence in the solution being delivered. We talked about how frequent deployments to the development environment enabled developers to test their changes, while less frequent deployments to the test environment allowed other parties to verify a potential production release. When everyone is happy, the production environment is updated, exposing the changes to end users.

The pillar of verifiable deployments describes the various techniques that can be used to verify a deployment when it reaches a new environment.

### General testing concepts

Testing is a nebulous term with often ill-defined subcategories. We will not attempt to provide authoritative definitions of testing categories here. Our goal is to offer a very high level description of common testing practices, and highlight those that can be performed during the deployment process.

#### What don't we test during deployments?

Unit tests are considered part of the build pipeline. These tests are tightly coupled to the code being compiled, and they must succeed for the resulting application package to be built. 

Integration tests may also be run during the build process to verify that higher level components interact as expected. The components under test may be replaced with a [test double](https://martinfowler.com/bliki/TestDouble.html) to improve reliability, or live instances of the components may be created as part of the test.

Unit and integration tests are run by the CI server, and any package that is made available for deployment is assumed to have passed all its associated unit and integration tests.

#### What can we test during deployment?

Tests that require a live application or application stack to be accessible are ideal candidates to be run as part of a deployment process.

Smoke tests are quick tests designed to ensure that applications and services have deployed correctly. Smoke tests implement the minimum interaction required to ensure services respond correctly. Some examples include:

* An HTTP request of a web application or service to check for a successful response.
* A database login to ensure the database is available.
* Checking that a directory has been populated with files.
* Querying the infrastructure layer to ensure the expected resources were created.

Integration tests can be performed as part of a deployment as well as during the build. Integration tests validate that multiple components are interacting as you expect. Test doubles may be included with the deployment to stand in for components being verified, or the tests may verify two or more live component instances. Examples include:

* Logging into a web application to verify that it can interact with an authentication provider.
* Querying an API for results from a database to ensure that the database is accessible via a service.

End to end tests provide an automated way of interacting with a system in the same way a user would. These can be long running tests following paths through the application that require most or all components of the application stack to be working correctly. Examples include:

* Automating the interaction with an online store to browse a catalog, view an item, add it to a cart, complete the checkout, and review the account order history.
* Completing a series of API calls to a weather service to find a city's latitude and longitude, getting the current weather for the returned location, and returning the forecast for the rest of the week.

Chaos testing involves deliberately removing or interfering with the components that make up an application to validate that the system is resilient enough to withstand such failures. Chaos testing may be combined with other tests to verify the stability of a degraded system.

Usability and acceptance testing typically require a human to use the application to verify that it meets their requirements. The requirements can be subjective, for example, determining if the application is visually appealing. Or the testers may not be technical, and so do not have the option of automating tests. The manual and subjective nature of these tests makes them difficult, if not impossible, to automate, meaning a working copy of the application or application stack must be deployed and made accessible to testers.

## Pillar 3. Seamless deployments {#seamless-deployments}

Deploying new versions of your applications inevitably involves taking the previous version offline and directing traffic to the new version. Ensuring end users are not adversely affected during this cutover requires some planning.

A logical solution is to perform public facing deployments at a time when end users are unlikely to be using the applications. If your customers are located in a similar timezone, deploying new versions of your application in the middle of the night effectively results in a seamless experience for end users.

Midnight deployments may not be glamorous, but if they *work for you*, this is a perfectly valid solution.

When downtime has to be kept to a minimum, or there is no good time for an outage window, some common deployment strategies can be used to seamlessly deploy new application versions.

### Seamless database deployments

No discussion on seamless deployments can begin without first addressing the issue of database updates.

A fundamental aspect of most seamless deployment strategies involves running two versions of your application side by side, if only for a short period of time. If both versions of the application access a shared database, then any updates to the database schema and data must be compatible with both application versions. This is referred to as backward and forward compatibility.

However, backward and forward compatibility is not trivial to implement. In the presentation [Update your Database Schema with Zero Downtime Migrations](https://www.youtube.com/watch?v=3mj6Ni7sRN4) (based on chapter 3 of the book [Migrating to Microservice Databases](https://developers.redhat.com/books/migrating-microservice-databases-relational-monolith-distributed-data)), Edison Yanaga walks through the process of renaming a single column in a database. It involves six incremental updates to the database and application code, and all six versions to be deployed sequentially.

Needless to say, seamless deployments involving databases require a great deal of planning, many small steps to roll out the changes, and tight coordination between the database and application code.

### Deployment strategies

There are multiple strategies to manage a cutover between an existing deployment and a new one.

#### Recreate

The recreate strategy does not provide a seamless deployment, but is included here as it is the default option for most deployment processes. This strategy involves either removing the existing deployment and deploying the new version, or deploying the new version over the top of the existing deployment.

Both options result in downtime during the period between the existing version being stopped or removed and the new version being started. However, because the existing and new versions are not run concurrently, database upgrades can be applied as needed with no backward and forward compatibility requirements.

#### Rolling updates

The rolling update strategy involves incrementally updating instances of the current deployment with the new deployment. This strategy ensures there is always at least one instance of the current or new deployment available during the rollout. This requires that any shared database must maintain backward and forward compatibility.

#### Canary deployments

The canary deployment strategy is similar to the rolling update strategy in that both incrementally expose more end users to the new deployment. The difference is that the decision to progress the rollout in a canary deployment is either made automatically by a system monitoring metrics and logs to ensure the new deployment is performing as expected, or manually by a human.

Canary deployments also have the option to halt the rollout and revert back to the existing deployment if a problem is discovered.

#### Blue/green deployments

The blue/green strategy involves deploying the new version (referred to as the green version) alongside the current version (referred to as the blue version), without exposing the green version to any traffic. Once the green version is deployed and verified, traffic is cutover from the blue to the green version. When the blue version is no longer used, it can be removed.

Any database changes deployed by the green version must maintain backward and forward compatibility, because even if the green version is not serving traffic, the blue version will be exposed to the database changes.

#### Session draining

The session draining strategy is used when applications maintain states tied to a particular application version.

This strategy is similar to the blue/green strategy in that both will deploy the new version alongside the current version, and run both side by side. Unlike the blue/green strategy, which will cut all traffic over to the new deployment in a single step, the session draining strategy will direct new sessions to the new deployment, while the existing deployment serves traffic to existing sessions.

After the old sessions have expired, the existing deployment can be deleted.

Because the current and new deployments run side by side, any database changes must maintain backward and forward compatibility.

#### Feature flags

The feature flag strategy involves building functionality into a new application version, and then exposing or hiding the feature for select end users outside of the deployment process.

In practice, the deployment of a new application version with flaggable features will be performed with one of the strategies above, so the feature flag strategy is a complement to those other strategies.

#### Feature branch

The feature branch strategy allows developers to deploy an application version with changes they are currently implementing, usually in a non-production environment, alongside the main deployment.

It may not be necessary to maintain database backward and forward compatibility with feature branch deployments. Because feature branches are for testing and tend to be short-lived, it may be acceptable that each feature branch deployment has access to its own test database.

## Pillar 4. Recoverable deployments {#recoverable-deployments}

The aim of implementing repeatable and verifiable deployments, tested in non-production environments before being released to production, is to identify bugs before they can affect end users. However, some bugs will inevitably find their way to production. When they do, it is important to restore the production environment to a desirable state.

### Rolling back or forward

Recovering from an undesirable deployment means rolling back to a previous good deployment, or rolling forward to a new version that returns the environment to a desirable state.

Either solution is suitable for stateless applications, but when a database is involved, rollbacks must be treated with care.

This is the [advice from the FlyWay project](https://flywaydb.org/documentation/command/undo#important-notes):

> While the idea of undo migrations is nice, unfortunately it sometimes breaks down in practice. As soon as you have destructive changes (drop, delete, truncate, …), you start getting into trouble. And even if you don’t, you end up creating home-made alternatives for restoring backups, which need to be properly tested as well.

[RedGate offers this advice for database rollbacks](https://documentation.red-gate.com/sca/developing-databases/concepts/advanced-concepts/rollbacks/rollback-guidance):

> Rather than investing time and energy into rollback planning, an alternative is to follow an approach that keeps you moving forward.

The blog post [Pitfalls with SQL rollbacks and automated database deployments](https://octopus.com/blog/database-rollbacks-pitfalls) has this advice:

> More often than not, the effort to successfully rollback a deployment far exceeds the effort it would take to push a fix to production.

When deployments involve database changes, I recommended that you roll forward to recover from an undesirable deployment.

### Rolling back

With repeatable deployments, rolling back can be achieved by rerunning a previous deployment. This is possible because the package versions, scripts, and variables are all defined by a repeatable deployment.

Rollbacks are also an explicit feature of several seamless deployment strategies:
* Canary deployments implement rollbacks by redirecting all traffic from the new deployment to the current deployment. 
* Blue/green deployments can rollback a deployment by cutting traffic back to the blue stack. 
* Session draining deployments can redirect new sessions to the current deployment, and optionally kill any sessions in the new deployment.

Rollbacks have the following benefits:
* A deployment issue can be fixed, without writing code, by rolling back to a previous deployment.
* A rollback leaves the system in a known, verified state.
* The time to complete a rollback can be measured in non-production environments.

Rollbacks have the following disadvantages:
* Rollbacks are all or nothing operations. You can not roll back individual features, only entire deployments.
* Rollbacks need to be tested as part of the deployment process to ensure they work as expected, which increases the complexity and time of the deployment process.
* If a rollback fails, it is likely that you will need to resolve the issue by rolling forward.
* Database rollbacks require special consideration to ensure data is not lost.

### Rolling forward

Rolling forward is simply another way to describe performing a new deployment. In this case the new deployment will only contain the fixes required to restore an environment.

Rolling forward has the following benefits:
* All deployments strategies, with or without a database, inherently support rolling forward.
* Teams gain experience in rolling forward with every deployment.
* You can choose the scope of a change or fix when rolling forward.
* Multiple deployments can be made in succession while rolling forward to resolve an undesirable deployment.

Rolling forward has the following disadvantages:
* Rolling forward typically requires a developer to implement a fix to include in the next deployment.
* Rolling forward may involve bypassing the environmental progression typically used to verify a deployment to get a fix deployed as quickly as possible.
* The production environment will be left in an undesirable state for as long as it takes to develop and deploy the next version.

## Pillar 5. Visible deployments {#visible-deployments}

Deployments are an abstract concept, and it is difficult to know what has been deployed where by inspecting the state of a system. Determining what versions of an application are installed from the files on a disk or the records in a database is like working out the mix of colors that were used to produce a tin of paint.

Being able to quickly view the current state of your environments is critical to understanding: 

* What features have been provided to your customers.
* What features are being tested. 
* What issues have been fixed.
* The history of any changes.

Listed below are a number of details required to gain full visibility into the state of your deployments.

### Commit messages

Commit messages capture the intention of source code edits, describing what changes were made and who made them. These messages can be invaluable when trying to understand at a low level what changes made it into a particular version of a package.

### Issue tracking

Often source code commits are made to resolve an issue documented in a dedicated issue tracker. These issues provide a space for bugs to be described, discussed, and tracked. Each issue is referenced by a unique identifier.

Capturing the issue IDs that relate to changes in a package version, and any deployment that includes that package version, provides insight into the issues that were resolved in any given deployment.

### CI build logs

A typical CI/CD pipeline will have a CI server that builds, tests, and packages an application. The log files for these builds contain a wealth of information such as which tests passed, which tests were ignored, which dependencies were included, and what packages were created. A link to the CI build from the deployment allows these log files to be quickly reviewed.

### Library dependencies

Almost every application deployed today is a combination of custom code and shared libraries, often provided by a third party. These libraries can be a source of bugs or security vulnerabilities, or provide new and useful features. Understanding the library dependencies that contributed to a deployment is important for auditing and debugging.

### Deployment versions

A release captures all of the above information, along with package versions, variable values, and scripts, in a release version. This release is then deployed to an environment.

Displaying which release versions are deployed to which environments provides a high level view of the state of your deployments. With this information anyone can see what was deployed where, and by drilling into the details of a release, can see the commit messages, issues, dependencies, and links to the CI builds.

## Pillar 6. Measurable deployments {#measurable-deployments}

Getting your software into the hands of your customers is the primary goal of any deployment process. To understand how well your deployment process is performing, you need to measure the metrics that define success for you.

Having measurable deployments means defining a set of common and agreed upon metrics, reliably collecting those metrics, and surfacing them in an easy to understand format.

Some common metrics that apply to deployments are:

* Deployment frequency: How frequently do you deploy to production?
* Lead time for changes: How long does it take for a commit to be deployed into production?
* Lead time for deployments: How long does it take for a release to be deployed to production?
* Time to recover deployment: How long does it take to successfully deploy a release after a failed deployment?
* Deployment fail rate: What is the ratio between failed deployments and successful deployments? 
* Change fail rate: What is the ratio between hotfix deployments and regular deployments? 
* Deployment duration: How long does each deployment take?

## Pillar 7. Auditable deployments {#auditable-deployments}

If the visible deployments pillar is about surfacing the current state of your environments and the changes made as part of the release, then auditing is about tracking who has been involved in the deployment process over time.

Auditing allows teams to see a history of deployments to environments, changes to deployment processes, changes to environments, who approved which deployments, and to determine the state of an environment at some point in the past.

In order for audit events to be useful, they must be able to be searched, filtered, exported, and reported on.

## Pillar 8. Standardized deployments {#standardized-deployments}

Just as repeatable deployments build confidence as a release is promoted across environments, standardizing on deployment processes across different projects allows teams to confidently leverage already proven processes.

There are two main aspects to standardizing deployments.

The first aspect is defining the initial deployment process used by the various deployments. This base deployment process can be a shared template that allows very specific settings to be customized. Or the entire process could be copied and pasted to bootstrap new projects, but otherwise allowing them to customize the process to their own needs.

The second aspect is defining who has the ability to edit a deployment process. By distinguishing between the ability to view, run, create, and edit a deployment process, teams can provide guarantees that only those individuals with the responsibility to create or edit deployment processes can do so.

## Pillar 9. Maintainable deployments {#maintainable-deployments}

Getting your deployments to the production environment is just the beginning. Diagnosing issues, collecting logs, performing backups, restarting services, rotating keys and testing connections are just some of the day to day operations tasks that keep your applications running and your customers happy.

While you could SSH or RDP into a server and start poking around, each change you make drifts your production environment from your non-production environments, making it just that little bit harder to implement repeatable deployments. It is also difficult to track the changes that were made, verify that they worked, and audit who changed what.

Like deployments, maintenance tasks should be repeatable, verifiable, visible, measurable, auditable, standardized and coordinated. Maintenance tasks represent the business knowledge required to keep your deployments running, and should be held to the same standards as your deployment processes.

## Pillar 10. Coordinated deployments {#coordinated-deployments}

Deploying a package into an environment is just one small part of the deployment process. Often deployments need to be coordinated with other business processes to ensure:

* The right people have given their approval.
* Interested parties are notified of the success or failure of a deployment.
* Deployments proceed in the correct order.
* Deployments can only occur during specific times.
* High priority deployments take precedence over low priority ones.
* Deployments are scheduled to take place at a predetermined time.
* Deployments can be triggered by external events.
* Deployments can trigger external events.

In practice a deployment process may be a single component in a wider ecosystem of business process management tools. The ability to orchestrate deployments from third party platforms and report results back allows teams to manage complex deployments as part of a broader business process.

## Conclusion 

Those are the ten pillars of pragmatic deployments that shape the [features](https://octopus.com/features) and philosophy of Octopus Deploy. I'm sure they will continue to evolve over time as new user cases emerge, but we believe they provide a solid foundation from which to keep building Octopus Deploy.

Happy deployments!
