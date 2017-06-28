---
title: "Octopus June Release 3.15"
description: This month's release ... 
author: matt.richardson@octopus.com
visibility: private
metaImage: 
tags:
 - New Release
---

## TODO: Add Header Image

This month's release ...

## In this post

!toc

## Release Tour

<iframe width="560" height="315" src="https://www.youtube.com/embed/TODO" frameborder="0" allowfullscreen></iframe>

## Let's Encrypt Integration

We're a bit fan of security here at Octopus, and we want to make it easy for you to be secure too. To that end, we've added support for automatically managing the SSL certificate used by the Octopus Portal, using [Let's Encrypt](https://letsencrypt.org). With a few simple steps, you can configure Octopus to register, request a certificate, and apply it to the Portal. Even better, it will automatically be renewed when the certificate approaches its expiry date, so you wont have to worry about manual renewals and manually re-configurating your Octopus Server.

## Feature 2

## OctoWatch iOS App

## Breaking changes

There are no breaking changes in this release, but it may be worth noting we have adjusted the SQL database schema upgrades as we discussed above.

`SQL Error 4060 - Cannot open database "OctopusDeploy" requested by the login. The login failed.`

If you see an error message like this after the installer completes, you can start the Octopus Server just like before and let it perform the schema upgrades.

## Upgrading

All of the usual [steps for upgrading Octopus Deploy](https://octopus.com/docs/administration/upgrading) apply. Please see the [release notes](https://octopus.com/downloads/compare?to=3.14.0) for further information.

## Wrap Up

That’s it for this month. We hope you enjoy the latest features and our new release. Feel free to leave us a comment and let us know what you think!  Happy deployments!
