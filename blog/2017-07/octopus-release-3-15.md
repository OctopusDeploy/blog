---
title: "Octopus June Release 3.15"
description: This month's release ... 
author: matt.richardson@octopus.com
visibility: private
metaImage: metaimage-release3-15.png
tags:
 - New Release
---

![Octopus 3.14 release announcement](blogimage-release-3-15.png)

This month's release ...

## In this post

!toc

## Release Tour

<iframe width="560" height="315" src="https://www.youtube.com/embed/TODO" frameborder="0" allowfullscreen></iframe>

## Let's Encrypt Integration

We're a big fan of security here at Octopus, and we want to make it easy for you to be secure too. To that end, we've added support for automatically managing the SSL certificate used by the Octopus Portal, using [Let's Encrypt](https://letsencrypt.org). With a few simple steps, you can configure Octopus to register, request a certificate, and apply it to the Portal. Even better, it will automatically be renewed when the certificate approaches its expiry date, so you wont have to worry about manual renewals and re-configurating your Octopus Server. If you've currently got your server on the internet over HTTP, it couldn't be easier - [move to HTTPS](https://octopus.com/docs/v/3.15/administration/lets-encrypt-integration) today.


## Allow un-tenanted projects to be deployed to tenanted machines 

During the development of multi-tenancy we decided to make a clear distinction between tenanted and un-tenanted deployment targets. As such we prevented un-tenanted projects from being deployed to tenanted machines. Our reasoning was essentially safety-first; we didn't want to ever leak tenanted deployments or variables.     

However this prevented a number of valid scenarios. For example deploying a common component (e.g. a telemetry service) to tenanted machines, or simply sharing a development server between tenanted and un-tenanted projects.    

_Note: this also applies to accounts and certificates, as they can also be scoped to tenants._

You let us know that we got this decision wrong (or at least incomplete). You told us via our support, via [UserVoice](https://octopusdeploy.uservoice.com/forums/170787-general/suggestions/16616209-allow-non-tenant-and-multi-tenant-deployments-to-t) and via the [GitHub issue](https://github.com/OctopusDeploy/Issues/issues/2722). 

So, as of Octopus 3.15, how machines, accounts, and certificates participate in tenanted-deployments is explicitly configurable. 

![Tenanted deployment configuration](tenanted-deployments-ui.png "width=500")


## OctoWatch iOS App

For iOS users who've been following along with [recent TLDR videos](https://www.youtube.com/watch?v=mZTLzcdHpwA&list=PLAGskdGvlaw39U9Ed9HhAHEr_AI3xNg56&index=8&t=569s), we have now released our first native iOS app, [OctoWatch](https://itunes.apple.com/us/app/octowatch/id1232940032?ls=1&mt=8) to the App Store.

With [OctoWatch](https://itunes.apple.com/us/app/octowatch/id1232940032?ls=1&mt=8), you can easily keep track of the status of your machines, the state of your releases and the tasks that are currently running, *across multiple Octopus Servers*.

![OctoWatch iOS app](octowatch-appstore.png "width=500")

[OctoWatch](https://itunes.apple.com/us/app/octowatch/id1232940032?ls=1&mt=8) is open-source and was designed as a weekend exercise in React-Native. If you're running Octopus and you find this app useful (or if you have any ideas on how to make this app more useful), please reach out and let us know. If you'd like to contribute any ideas, we'd be happy to review a [pull request](https://github.com/OctopusDeploy/OctoWatch).

## Console Improvements

## Breaking changes

There are no breaking changes in this release, but it may be worth noting we have adjusted the SQL database schema upgrades as we discussed above.

`SQL Error 4060 - Cannot open database "OctopusDeploy" requested by the login. The login failed.`

If you see an error message like this after the installer completes, you can start the Octopus Server just like before and let it perform the schema upgrades.

## Upgrading

All of the usual [steps for upgrading Octopus Deploy](https://octopus.com/docs/administration/upgrading) apply. Please see the [release notes](https://octopus.com/downloads/compare?to=3.14.0) for further information.

## Wrap Up

That’s it for this month. We hope you enjoy the latest features and our new release. Feel free to leave us a comment and let us know what you think!  Happy deployments!
