---
title: "Octopus January Release 2018.1"
description: What's new in Octopus 2018.1
author: shaun.marx@octopus.com
visibility: private
metaImage: metaimage-shipping-2018-1.png
bannerImage: blogimage-shipping-2018-1.png
published: 2018-01-21
tags:
 - New Releases
---

This January release of Octopus Deploy provides a number of security enhancements such as being able to run tasks on on the Octopus Server as a different user account. You'll also notice our new versioning strategy: what would normally have been Octopus `4.2` is actually `2018.1`.

## In this post

!toc

## We've changed our versioning strategy

Read more about [why we changed](/blog/2018-01/version-change-2018.md) which also dives into our evolution as a software product company.

## History: Running tasks on the Octopus Server

Prior to Octopus `3.0` you required a Tentacle somewhere to do work as part of a deployment. If you wanted to deploy a web site to Azure, you would need to configure a Tentacle as a deployment target and use it as a jump box, when all you needed to do was push your package to Azure and call some APIs. That Tentacle wasn't really a deployment target at all.

In Octopus `3.0` we added the concept of a **worker** into Octopus Server. It's not a concept we've talked about much to date because it's an underlying technology that's built into Octopus Server and just does what it says on the box: work. Now you could install Octopus Server and deploy a web site to Azure in minutes all without any Tentacles involved.

In Octopus `3.3` we added a feature which lets you [run deployment steps on the Octopus Server](https://octopus.com/docs/deployment-process/how-to-run-steps-on-the-octopus-server) which again removed a lot of friction for scenarios other than deploying to Azure.

However, this convenience came at a cost: security. By default Octopus Server runs as the highly privileged `Local System` account on Windows. We typically recommend running Octopus Server as a different account, either a User or Managed Service Account (MSA), so you can grant specific privileges to that account.

## Now: Running tasks on the Octopus Server as a different user

In Octopus `2018.1` you can configure the built-in worker to execute tasks as a different user account. This user account can be a down-level account with very restricted privileges.

```plaintext
Octopus.Server.exe builtin-worker --username=OctopusWorker --password=XXXXXXXXXX
```

All tasks which execute on the Octopus Server will run as that user account. The only gotcha is that the user account running the Octopus Server needs the correct privileges to launch processes as the built-in worker user and impersonate the built-in worker user.

We outline these requirements and how to configure your Octopus Server in our documentation TBD!

## Future: External workers

We [talked about workers briefly in our 2018 roadmap](roadmap-2018#scalability-and-continual-improvements) and will be talking in more detail with you all about this in the near future. In summary, workers will be an evolution of `Run on Server` which will let you delegate tasks in a deployment to a worker which may be external to your Octopus Server. We still plan to ship a built-in worker which you can configure as we described earlier, or disable it altogether as you see fit.

## Breaking Changes

We've made two behavioral changes to Octopus which may effect certain customers.

* [Package acquisition can sometimes be too parallel](https://github.com/OctopusDeploy/Issues/issues/3974)
* [Auto machine removal timing does not take into account if Octopus Server is offline](https://github.com/OctopusDeploy/Issues/issues/3924)

## Upgrading

As usual [steps for upgrading Octopus Deploy](https://octopus.com/docs/administration/upgrading) apply. Please see the [release notes](https://octopus.com/downloads/compare?to=2018.1.0) for further information.

## Wrap up

That’s it for this month. We hope you had a fantastic festive season and find the new features useful. Feel free leave us a comment and let us know what you think! Go forth and deploy!