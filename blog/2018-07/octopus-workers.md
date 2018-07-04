---
title: Octopus Workers
description: Octopus now supports external worker machines
author: michael.compton@octopus.com
visibility: private
metaImage: ??
bannerImage: ??
published: ??
tags:
 - Workers
---

**top image**

In 2018.7 we introduced a new feature we call Workers.  In this post, I'll tell you the what and why of workers and how to use workers to move steps off the Octopus Server.  Following posts give further walk-through examples for scalling with workers and using workers for cloud deployments.

Workers allow some nice ways to set up your deployments and move work off your Octopus Server, so it's well worth reading carefully, but we've also designed it [so you never even need to know it's there](#What-wait-No-Now-theres-workers-and-pools-and-a-built-in-worker-I-don-t-care-just-let-me-have-it-back-to-how-it-used-to-be) if you don't want to use it.

## What are these workers anyway?

Since version 3.0, Octopus has had one worker.  We didn't call it a worker to start with, and you might have used it without even knowing it was there.  It's called the built-in worker, and it's bundled with the Octopus server.

Azure, AWS and Terraform steps all need somewhere to run, so, out-of-the-box that's the built-in worker on the Octopus Server.  Steps in Octopus are executed by [Calamari](https://github.com/OctopusDeploy/Calamari), our open-source, conventions-based deployment tool.  Lots of the time, Calamari runs on a deployment target.  But in the case of Azure, AWS and Terraform steps, the Octopus Server uses the built-in worker to invoke Calamari locally.

The give-away that the server can invoke Calamari is script steps.  Those call out the option of running the step on the Octopus server.

**screenshot of a script step in here**

All a worker does is takes these points where the server could execute Calamari locally using the built-in worker and gives you the option of executing on a worker somewhere else.

### What a worker is and isn't

A worker doesn't take over orchestrating a deployment in place of the server; it just executes steps that the server instructs it to.  Nothing has even changed about how script, Azure, AWS or Terraform steps are executed; workers  just provide an option about where those steps are executed.

The Octopus Server orchestrates the whole deployment process, deployment targets are where your deployment goes, and workers are machines that can execute some steps for the server but aren't the target of the deployment.  

So, workers are just machines that can run script, Azure, AWS and Terraform steps, and can be listening tentacles, polling tentacles or SSH machines (SSH workers can only run bash scripts).

## When might you want to use a worker

Over the course of a couple of posts I'll flesh out the details of three cases where workers come in handy (or are essential).

1. Moving steps off the Octopus Server (security)
1. Workers for scaling up (performance)
1. Setting up cloud workers (cloud)

Of course, there are also other ways to use workers.

But before we get into the examples, let's look at how the whole workers setup works.

## How workers _works_

**drawn octopus pic in here of workers in pools**

### Workers

Workers are listening tentacles, polling tentacles or SSH machines.  The setup is the same as for tentacle or SSH deployment targets.  Workers even use the same Tentacle and Calamari binaries as deployment targets.

**...walk through of an example worker setup with screen shots**

### Worker Pools

Workers are grouped into worker pools.  You might have a single pool, or multiple pools for workers of different capabilities or loaded with different libraries.

**screen shots, comments about pools etc**

### Running steps on workers

Steps - well, script, Azure, AWS or Terraform steps - can now target a pool and will be executed on a worker from that pool.  There's a default pool that steps are assumed to target if nothing else is specified.

Just two rules govern how Octopus decides where to execute a step that requires a worker.  Octopus will run the step on...

1. the built-in worker, if the step resolves to the default pool and there are no workers in the default pool, or
2. any healthy worker from the given pool, otherwise.

That's pretty much it: you setup workers (as easy as setting up tentacles), group the workers into pools (as easy as putting a deployment target into an environment), and then you point a step at the pool and Octopus distributes out the work of your deployment process.

## What, Wait, No.  Now there's workers and pools and a built-in worker, I don't care, just let me have it back to how it used to be

No worries, we've got you covered.  We took a hard look at use cases for not using workers and for transitioning to workers.  We think we got smooth answers for both.

**If you don't want to use workers, then it's really simple - just ignore it**.  If your an existing Octopus user, your steps won't change, there's no changes to any of your deployment processes and your Octopus experience won't change.  

Point (1) above says it all.  Any steps that would require a worker will resolve to the default worker pool, because your steps won't say any different, and that will end up at the built-in worker, which is the same experience Octopus users have had since version 3.0. 

## Stopping steps running on the server

We've also got a nice story for the transition away from runnings steps on the Octopus server.  No deployment processes need updating, just a tiny bit of setup and Octopus will move steps off the server and onto workers.

So you love all your Devs, but would much rather if they couldn't run code on your Octopus Server.  Well, all your existing Azure, AWS and Terraform steps (and any script steps targeted at the server) won't mention a worker pool and so end up with the (empty) default pool and thus the built-in worker.  But, if you drop even one worker in that default pool, then rule (2) applies and the step runs on that worker and not the built-in worker.

Let's have a look at that in action.  I started with an Octopus setup with no workers (other than the built-in worker, of course) and no worker pools (other than the default pool). I created a simple project with a script targeted at the server.

**Screen shot of step**

After deploying the project, the logs clearly point out that the script ran on the Octopus Server.

**logs screen shot**

I then provisioned a tentacle and registered it as a worker in the default pool.

**Screen shot of worker pool infrastructre page**

On deploying the same project (unchanged), the logs let me know it ran on the worker instead of the server.

**screen shot of logs**

That's all it takes.  Just one tentacle is enough to stop user code executing on the Octopus Server.  

Of course it's also possible to provision multiple workers to share the work the server hands out.

It's also possible to turn off the built-in worker, meaning that it never gets invoked.  Check the option on the features page.

The same concurrency rules apply as always did.  The server still respects your parallel steps and `Octopus.Action.MaxParallelism`, so multiple concurrent steps could be running on workers, even the same worker, just as the built-in worker runs many steps concurrently.


...wrap up for this post...


## ...the following in other posts
.... then some walk-through examples that go from nothing to the whole thing worker in 2 other posts ?  or one deep dive post?....

## Scalling up

using multiple workers to move say AWS/Azure steps off the server and thus reduce server load

## Setting up my clouds just how I like them

setting up cloud workers with their own default pools so you can play fun games with cloud deployments ... example of multi region Azure deployment with polling workers in each locked-down region.

