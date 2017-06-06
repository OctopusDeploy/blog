---
title: "Octopus June Release 3.14"
description: TODO
author: nick.josevski@octopus.com
visibility: private
metaImage: metaimage-release3-14.png
tags:
 - New Release
 - TODO
---

![Octopus 3.14 release announcement](blogimage-release-3-14.png)

This month's release brings some exciting new features including **TODO**, among other things!

## In this post

!toc

## Release Tour

<iframe width="560" height="315" src="https://www.youtube.com/embed/TODO" frameborder="0" allowfullscreen></iframe>

## Introducing A

TODO

## Certificates are now generated using SHA256

As promised in our blog post on [shattered](http://octopus.com/blog/shattered), we have updated our certificate generation to use SHA256, rather than the previous SHA1. This ensures any new installations or regenerated certificates will use SHA256, but will not affect any existing certificates. We will be rolling out features in the near future to make it easier to replace older certificates.

## Tentacle split

Until now we have always released a new version of Tentacle with every version of Octopus Server. Today we split the Tentacle!

### Previous Tentacle cadence

With each release of Octopus Server we also released a version of Tentacle. As a result, every time you upgrade your Octopus Server you have been prompted to upgrade all of your Tentacles. In the majority of our releases we have made no changes to Tentacle.

### The new Tentacle cadence

A new version of Tentacle will only be released when the Tentacle has changed. At this stage changes will be signified by a bump in the Tentacle version. Octopus Server will bundle the version of Tentacle that was most recently released and use it for Tentacle upgrades.

### What does this mean for you?

Less Tentacle upgrades! Since the release of Octopus 3.0 there have been a few small tweaks to Tentacle, mostly adding new commands and minor changes to the communication stack. Octopus Server v3.x is compatible with all Tentacle 3.x versions. We hope splitting the Tentacle helps relieve some of the hassle and friction involved with upgrading Octopus and provides better communcation about changes to Tentacle.

### Coming soon

We aim to give Tentacle and some of our other open source repositories their own release notes to save you having to sift through the Octopus Server release notes to find items of interest. We will also give you the ability to automatically upgrades your Tentacles as soon as a new Tentacle version is released rather than having to wait for the next Octopus Server release.

## Upgrading

All of the usual [steps for upgrading Octopus Deploy](https://octopus.com/docs/administration/upgrading) apply. Please see the [release notes](https://octopus.com/downloads/compare?to=3.14.0) for further information.

## Wrap Up

That’s it for this month. We hope you enjoy the latest features and our new release. Feel free to leave us a comment and let us know what you think!  Happy deployments!
