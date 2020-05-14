---
title: Re-thinking feature branch deployments
description: Feature branches should be tested prior to merging into master.  All too often companies have a single static test environment.  
author: bob.walker@octopus.com
visibility: public
published: 2020-12-31
metaImage: 
bannerImage: 
tags:
 - Engineering 
 - Database Deployments
---

I transitioned to Git in 2013.  Since that time I have been doing feature branch testing all wrong.  The problem was I've worked in places with the same static environments, `Dev` -> `Test` -> `Staging` -> `Production`.  Each environment had one instance of my application.  It reflected what was in the `master` branch.  The only way for QA to test a new feature was to merge code into `master`.  What I should've been doing is standing up a sandbox for the feature branch for QA to test.  The `Dev` -> `Test` -> `Staging` -> `Production` lifecycle represented my pre-git life.  In this article I will walk through how I've adjusted my thinking to better leverage git.

!toc

## Deployment lifecycles

I have been busy configuring an application to demo feature branches.  I started a two lifecycles along with corresponding channels.  These lifecycles represented the same process I have been using since I started being a developer back in 2004.

- Default: `Dev` -> `Test` -> `Staging` -> `Production`
- Feature Branch: `Dev` -> `Test`

The intended development workflow is:
1. All feature branches go to the `Dev` environment and if that was successful then `Test`, for QA / Business Owners to verify the work.  New infrastructure is spun up for that feature branch only.  The branch name is stored in the pre-release tag.
2. Once the feature branch looks okay, a PR will merge the feature branch into master.  Unlike feature branches, there are no pre-release tags for this package. That’ll deploy the merged code to the static `Test` database.  The feature branch infrastructure is then backed up and deleted.
3. QA and the Business Owners will verify everything still looks good in the static `Test` environment.  
4. The release would be pushed to `Staging` and reverified.
5. Assuming everything looks good, that release will be promoted to `Production`.

As I got deeper into setting up the example for this post the same question kept popping into my head.

> Why would I re-deploy to dev and test  after merging to master?  Why do those environments have static resources , such as databases, websites and file storage?

## Static vs dynamic environments

A static environment is an environment where application specific resources (databases, file storage, etc) have to be static.  Spinning up and using new resources requires a lot of coordination and communication to avoid an outage or downtime.  Any sort of downtime which affects non IT Staff (external users, business users, etc) is consider a BIG DEAL.  Deployments to these environments are measured in days not minutes.

Most applications deployed via Octopus Deploy use RDMS databases such as SQL Server, Oracle, PostgreSQL, and MySQL.  The `Production` instance of the database has to be static.  It isn't feasible to create a new copy of a `Production` database for each deployment.  As cool as it would be, the time and resource (disk space, cpu usage, RAM usage) cost would be too high.  One application deployment to `Production` a week is considered fast.  Typically it is once a fortnight to once every eight weeks.

Based on those reasons `Production` is a static environment.  

It is common for companies to have an "Production-Like" environment to run final tests prior to going to `Production`.  These environments have names like `Staging`, `Pre-Prod`, `Integration`, or `UAT`.  For the purposes of this article I will use `Staging` to represent those environments.  

`Staging` is as "Production-Like" as possible.  This includes making websites publicly available to external users to test out upcoming features.  Even if it is not publicly available, non IT Staff, may access it to try out new features prior to going to `Production`.  `Staging` is deployed to more often than `Production`, perhaps once or maybe twice a day.

Based on the static environment criteria, `Staging` a static environment as well.

A dynamic environment is an environment where application specific resources are in flux.  Resources can be spun up and down on as needed.  Or, there is bug in the authentication pipeline.  Or, a person is doing a deployment and it hasn't finished yet.  Down time, while annoying, is tolerated.  Downtime is limited to IT Staff.  Deployments occur multiple times a day.

`Test` is very much a dynamic environment.  

## The downside of static test environments

When I think of `Dev` and `Test` environments, I think of one word: *churn.*

### Test environments are rarely stable

I've worked on projects where at the height of crunch time we did 20-25 deployments to `Test` in a 10 hour span.  That is a deployment every 24 - 30 minutes.  We were doing 1.5x as many deployments to `Dev`.  We had a single `Dev` instance and a single `Test` instance of the application.  When it was time for QA to test new features we would merge our changes into master to get it deployed to `Test`.  Believe it or not, we weren't perfect on the first try.  

Those 20-25 deployments are actually 4-5 deployments per feature.  Some deployments were because of bugs.  Some deployments were because of a incorrect merge conflict resolution.  The problem was features were half-done before merging into master. It was ware when more than 1 feature was finished on the same day.  Planning when to release code was a bit of a nightmare.  Prior to a scheduled release we would "freeze" feature work and several days of "bug bashing."  A freeze meant no new features could be merged into master.

We had a whiteboard in our team area indicating if it was okay to merge into master.

This also made it very difficult to deploy hotfixes.  We had to create a separate channel in to bypass `Dev` and `Test` and go straight to `Staging`.  

### Very few apps are completely isolated

With the advent of SOA and Microservices, applications are no longer these big massive (isolated) monoliths.  Before each application would have their own logging, authentication, configuration, and notification modules.  That's...okay when you have one application.  Once you add two or more applications into the mix the cost of maintaining those duplicate modules grows exponentially.  Thankfully companies wised up to the duplicate effort and now they have a single logging server, or a single authentication service and so on.

This leads me to my next point.  Applications, it seems more than ever, are dependent on other applications to function properly.  Services my application is dependent are being changed at the same time.  If a dependent service goes down due to a bug, my application enters into a degraded state. Certain functionality is limited until that dependent service is back up.

We do this because we point our applications to one in other when in the same environment.  It makes sense when every environment is static.  My app is in `Test`, a dependent service is in `Test`.  The URL of that service never changes, I should point my application to that.

### What is the point of the dev environment?

Most places I worked the `Dev` environment was rarely used.  It didn't have very much data to test with.  It was almost always down.  No one really used it.  It was just...there.  Some teams would run automated tests against it.  But the results were typically ignored because the data was so out of date.  If a deployment failed we would band-aid a solution so it could get to the `Test` environment so QA can continue working.  

`Dev` was always a rubber stamp environment.  

## Re-thinking environments for feature branch deployments

`Production` and `Staging` aren't going anywhere.  Nor should they.  `Dev` and `Test` need to be completely re-thought.

- Combine `Dev` and `Test` into one environment, `Test`.
- When a new feature branch is checked in, new infrastructure for that feature branch is spun-up in `Test`.  
- Subsequent check-ins for that feature branch will go to that infrastructure.
- By default, all applications in `Test` use the `Staging` instance of their dependent applications.  This can be overwritten to point to `Test` when work on 1 to N applications are tightly coupled.
- When the feature branch is merged into master the testing infrastructure is torn down.  The deployment pushes code to `Staging`.
- After final verification and sign-off the code is pushed to `Production`.

The lifecycles will be:

- Default: `Staging` -> `Production`
- Feature Branch: `Test`

Unfinished code will no longer be merged into master.  This will make scheduling a release much easier.  What is in master has been signed off by QA and any Business or Product Owners.  You could merge Feature A into master on Monday, deploy on Tuesday, then merge Feature B into master on Wednesday and deploy on Friday.  Each deployment will be much smaller as well.

The same holds true for bug fixes.  They would be treated as feature branches.  When a bugfix branch is check in, new infrastructure will be stood up to verify the fix actually fixes the issue.  Once it is ready to go it is merged into master and pushed up to `Staging`.  

If you are using Gitflow, you could configure the build server to kick off releases to `Staging` when a change is checked into a release branch.  

## External Services

In this configuration the `Staging` environment matches `Production` as close as possible.  Which means it should be stable.  The only time `Staging` is different than `Production` is prior to a `Production` deployment.

As a rule of thumb, all the services in `Test` which are dependent on external services should point to `Staging`.  There are cases where multiple applications are being changed for a release.  The changes are all tightly coupled together.  In that specific instance the applications would point to specific feature branch services on `Test`.  Essentially, it is a sandbox of sandboxes.

## New infrastructure vs re-using existing infrastructure

This is a question which you will have to answer.  When you spin up a sandbox will you spin up new infrastructure or re-use existing infrastructure?  Spinning up new infrastructure has the benefit of making it much easier to tear down when finished.  Especially if you are using AWS, Azure or GCP.  

However, new infrastructure has a cost associated with it, be it running on the cloud or self-hosted.  For the cloud, the cost is very much real in what you are billed each day.  For self-hosted, there is a time concern plus resource utilization.  For example, does the SAN have enough space for another virtual disk, or is there enough v-cpus available in the hypervisors?  In general, most servers in `Test` are not being fully utilized.  Even when QA is testing, if you look at a resource monitor the CPU usage is probably under 20%.  And most web servers, database servers, and app servers allow you to host more than one application, website or database.  

There isn't a right or wrong answer to this.  It is a "do what is best for you, your sanity, and your company."

## Feature branch databases and testing data

It's easy to spin up a new empty database.  The tricky part is populating that empty database with data.  In most places I've worked `Staging` was a sanitized copy of `Production`.  Or `Staging` was populated with lots of test data.  In either case, when spinning up a new sandbox it should restore a copy of the `Staging` database.  That is much easier said than done.  That is where I would start at least.  

## Leveraging Octopus Deploy

I am working on a follow-up post to this article that will walk through a full-on setup of a build server and Octopus Deploy.  That is out of scope for this article.  There is already a lot to take in which this article, I didn't want to muddy the waters with a full walk-through as well.

In general the solution will use the following features within Octopus Deploy.

- [Runbooks](https://octopus.com/runbooks) -> This will be used to spin up and tear down the sandbox for a feature branch.
- [Channels](https://octopus.com/docs/deployment-process/channels) -> Channels allow for different lifecycles in a deployment process.
- [Prompted Variables](https://octopus.com/docs/projects/variables/prompted-variables) -> Allows specific settings (database name, urls, external references, etc.) to be overwritten when using a feature branch.

## Conclusion

You'll often see the phrases "branches" and "trunks" when reading about git workflows.  The idea being a branch is short lived, after a while it either merges back to a trunk or it gets pruned.  The trunk is long living, it is what branches are, um, branched off of.

This article proposed treating your testing environments like branches and the staging/production environments like a trunk.  It is a mind-shift.  And it is much easier to talk about this rather than actually doing it.  As I said in the previous section, in my next post I will provide a guide on configuring CI/CD pipeline using this new workflow.  It will include both the build server and Octopus Deploy.

Until next time, Happy Deployments!