---
title: Demystifying workers
description: Learn about workers and how they work.
author: shawn.sesna@octopus.com
visibility: private
published: 2022-06-16-1400
metaImage: 
bannerImage: 
bannerImageAlt: 
tags:
 - 
---

As the Octopus Deploy product evolved, more and more demands were being made on the server in which it was installed.  In those early days, any step that didn't execute directly on a [Target](https://octopus.com/docs/infrastructure/deployment-targets) were executed on the server itself.  To address the growing list of tasks executing directly on the server, Octopus came up with the concept of workers.  In this post, I'll address some common questions and misconceptions about workers and how they operate.

## What exactly is a worker?
In essence, a worker is a tentacle.  It runs the same tentacle software as a deployment target, however, is registered with the server in worker pools.  Worker pools are a collection of worker machines.

### If a worker is a tentacle, does it count as a target for licensing?
Despite running the tentacle software, workers are viewed as an extension of the Octopus Server and are therefore not counted as targets.  Under the current licensing model, there is no limit to how many worker machines you can have.

## How can I use a worker?
When defining a step in a [Runbook](https://octopus.com/docs/runbooks) or [Project Deployment Process](https://octopus.com/docs/projects/deployment-process), you are able to tell Octopus that this step will run on a worker and select a pool.

![](octopus-step-worker-pool.png)

Workers can be used with steps that only require connection string and don't _need_ the files contained within the package on the server being deployed to.  The most common use cases are database deployments, API calls, or running scripts.    

Another example is deployment of SQL Server Reporting Services (SSRS) reports.  Deployments to an SSRS server are done by making calls to Web Services that it hosts which means that the tentacle (worker) doesn't need to be on the SSRS machine at all.

### Worker pool variable
The keen eyed observer would have noticed there is a second selection for the `Worker Pool` section, `Runs on a worker from a pool selected via a variable`.  The [worker pool variable](https://octopus.com/docs/projects/variables/worker-pool-variables) was created to solve issues where customers needed a different worker pool for different situations, such as environments.  Some customers have security segregated in such a way that workers in Development were not allowed to touch resources in Test.  Using a worker pool variable, you can scope pools to environments or even [Tenant Tags](https://octopus.com/docs/deployments/patterns/multi-tenant-deployments/tenant-tags) denoting things like specific Azure regions

![](octopus-worker-pool-variable.png)

### Kubernetes deployments
Kubernetes (K8s) deployments are the only target type that require the use of workers.  When creating a K8s deployment target, you have the ability to select a specific pool to use during Health Check operations.  

Deployments to K8s interact with an API, providing instructions to the K8s cluster rather than deploying files directly to it.


### Execution containers
A byproduct of the evolution of Octopus is the increasing number of technologies supported by the tool.  As this list continued to grow, we were required to bundle more and more software components into Octopus.  For various reasons, some customers were unable to keep up with the release cycle of Octopus and encountered issues of older versions of our software, but needing modern versions of the bundled software.  In addition, not all projects in an Octopus instance may be using the same version of the sofware so installing directly on a worker would introduce a different set of problems.  The answer to both the bloating of Octopus and version incompatibility is the ability for the worker to launch a container to perform the work.  If the worker machine has Docker installed, they can launch containers of different version to cover the difference scenarios listed above.

## How do workers execute?
If you've ever attempted to execute two deployments against the same target machine, you may have noticed that the deployments seem to bounce back and forth between the tasks, executing one step at a time.  This behavior is by design to protect the target from multiple deployments attempting to update the same resource at the same time, such as an IIS metabase.  Workers, on the other hand, are configured to be able to handle multiple tasks simultaneously.

:::information
Activities such as `Acquire Packages` will result in a worker being locked and any other deployment/runbook using the same worker will be in a wait state.
:::

### How is a worker selected from the pool?


A worker is selected from the pool at the beginning of a deployment or runbook run.  The same worker will be used for the entire deployment process, the exception being when a [Manual Intervention](https://octopus.com/docs/projects/built-in-step-templates/manual-intervention-and-approvals) step is encountered and the task is removed from the queue.  Once the task has been placed back into the queue, a worker is selected again to continue processing.

#### If I have parallel steps, will worker selection round-robin?
It is a common misconception that once a worker has completed a task, it will select a new worker from the pool for the next task in a deployment/runbook that uses the pool.  As previously stated, worker selection is done at the beginning of the deployment/runbook and is used throughout the task.

#### I'm using Octopus Cloud, how are dynamic workers selected?
Octopus Deploy maintains a set of workers that customers can use as dyanamic workers available in the following pools:
- Default worker pool (Windows Server 2016)
- Hosted Windows (Windows Server 2019 with Docker)
- Hosted Ubuntu (Ubuntu 18.04 with Docker)

Each cloud instance can lease one worker per pool for a two hour period.  This means all steps that use workers will use the same worker for all tasks until the two hours has elapsed at which time a new worker will be leased.  Once the two hours has elapsed, the worker machine will be destroyed and a new worker machine will be created to take its place amongst the machines available to customers.