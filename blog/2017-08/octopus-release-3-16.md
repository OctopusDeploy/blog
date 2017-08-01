---
title: "Octopus August Release 3.16"
description: This month's release includes ...
author: rob.pearson@octopus.com
visibility: private
metaImage: metaimage-release3-16.png
tags:
 - New Releases
---

![Octopus 3.16 release announcement](blogimage-release-3-16.png)

This month's release includes ... 

## In this post

!toc

## Release Tour

<iframe width="560" height="315" src="https://www.youtube.com/embed/TODO" frameborder="0" allowfullscreen></iframe>

## SSH Targets sans Mono 

Octopus Deploy supports deploying to Linux and MacOS via [SSH Targets](https://octopus.com/docs/deployment-targets/ssh-targets).

Because [Calamari](https://octopus.com/docs/api-and-integration/calamari) (the Octopus deployment executable) is built with .NET, [Mono](http://www.mono-project.com/) was required to be installed on SSH Target servers.

As of Octopus 3.16, Mono is no longer required!

SSH Targets now provide an option to use a self-contained Calamari.   

![SSH Target .NET Settings](ssh-mono-not-installed.png "width=500")

The self-contained Calamari is built with .NET Core 2.0.

Because .NET Core 2.0 is currently a preview release, we felt obliged to mark this feature as _beta_.  But be assured, this is a fully supported feature.  We believe that .NET Core will provide the foundation of Octopus Deploy's cross-platform support in the future, and we will likely at some point deprecate the Mono-based option. 


## Feature 2

## Let's Encrypt required update

Let's Encrypt recently deployed an update, returning more data from a specific API call. Unfortunately the library we use to communicate with Let's Encrypt was unable to handle this, meaning both new registrations and renewals were unable to complete successfully. If you have setup the Let's Encrypt integration, please upgrade to ensure that your Portal certificate renews correctly.

## Breaking changes

There are no breaking changes in this release.

## Upgrading

This release contains a post-install data fix that may take some time (depending on the size of your Events table), so please ensure you allow time for this to complete. If you are running the [watchdog service](https://octopus.com/docs/administration/service-watchdog), please ensure this is stopped during the upgrade.

All of the usual [steps for upgrading Octopus Deploy](https://octopus.com/docs/administration/upgrading) apply. Please see the [release notes](https://octopus.com/downloads/compare?to=3.16.0) for further information.

## Wrap up

That’s it for this month. We hope you enjoy the latest features and our new release. Feel free to leave us a comment and let us know what you think!  Happy deployments!
