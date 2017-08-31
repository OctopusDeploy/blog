---
title: "Octopus September Release 3.17"
description: This month's release includes first-class Java deployment support, Swagger support for the Octopus API bug fixes and more.
author: rob.pearson@octopus.com
visibility: private
metaImage: metaimage-release3-17.png
tags:
 - New Releases
---

![Octopus 3.17 release announcement](blogimage-release-3-17.png)

This months release is big and we're very proud to announce ... `Rob to finish intro`.

## In this post

!toc

## Release Tour

<iframe width="560" height="315" src="https://www.youtube.com/embed/TODO" frameborder="0" allowfullscreen></iframe>

## First-class Java deployments

3.17 introduces a number of new steps for deploying and managing applications against Java application servers, as well as providing support for managing `jar`, `war`, `ear` and `rar` files in the built-in Octopus library.

![Java Steps](java-steps.png)

These new steps allow Java applications to be deployed to WildFly 10+ and Red Hat JBoss EAP 6+ application servers, as well as Tomcat 7+. In addition, the `Deploy Java Archive` step allows Java applications to be copied to a custom location on the target machine, allowing Java applications to be deployed in any Java application server capable of using file copy deployments.

See the [documentation](http://g.octopushq.com/JavaAppDeploy) for more information on these new steps.

## Octopus API Swagger Support

`TODO: Cam`

## User administration and authentication performance improvements

`TODO: Shannon`

## Breaking changes

There are no breaking changes in this release.

## Upgrading

All of the usual [steps for upgrading Octopus Deploy](https://octopus.com/docs/administration/upgrading) apply. Please see the [release notes](https://octopus.com/downloads/compare?to=3.17.0) for further information.

## Wrap up

That’s it for this month. We hope you enjoy the new features and our latest release. Feel free to leave us a comment and let us know what you think! Happy deployments!
