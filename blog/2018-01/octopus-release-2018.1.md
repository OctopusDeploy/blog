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

This January release of Octopus provides a number of security enhancements such as being able to run tasks on on the server as a different user. You'll also notice our new versioning strategy: what would normally have been Octopus `4.2` is actually `2018.1`.

Read more about [why we changed our versioning strategy](/blog/2018-01/version-change-2018.md).

This provides an additional level of security ensuring tasks can't do anything they aren't supposed to. In addition there are a number of changes which we are including as part of this update in preparation for [Octopus in the cloud](https://octopus.com/cloud/register-interest).

## In this post
!toc

## Run on Server account


## Breaking Changes

* [Package acquisition can sometimes be too parallel](https://github.com/OctopusDeploy/Issues/issues/3974)
* [Auto machine removal timing does not take into account if Octopus Server is offline](https://github.com/OctopusDeploy/Issues/issues/3924)

## Upgrading

As usual [steps for upgrading Octopus Deploy](https://octopus.com/docs/administration/upgrading) apply. Please see the [release notes](https://octopus.com/downloads/compare?to=4.2.0) for further information.

## Wrap up

That’s it for this month. We hope you had a fantastic festive season and find the new features useful. Feel free leave us a comment and les us know what you think! Go forth and deploy!