---
title: Octopus Deploy 2019.11 - Painless Jenkins integration
description: Octopus 2019.11 - Painless Jenkins integration
author: rob.pearson@octopus.com
visibility: private
published: 2019-12-02
metaImage: octopus-2019.11-release-image.png
bannerImage: octopus-2019.11-release-image.png
tags:
 - Product
---

<iframe width="560" height="315" src="https://www.youtube.com/embed/TODO" frameborder="0" allowfullscreen></iframe>

**Octopus Deploy 2019.11** has shipped and it brings first-class Jenkins plugin to make it painless to integration Jenkins builds with Octopus deployments. This brings the Jenkins plugin to feature parity with our [TeamCity](TODO LINK), [Azure DevOps](TODO LINK) and [Bamboo Server](TODO LINK) plugins.

<h2>In this post</h2>

!toc

## Painless Jenkins Integration

![Octopus Jenkins Plugin](octopus-deploy-jenkins-plugin.png "width=600")

Octopus has had a Jenkins community plugin for years built and maintained by [Brian Adriance](https://github.com/badriance) and other contributors. Octopus has worked with Brian to take over the plugin and make it officially support by Octopus. We're indebted to the effort from Brian and other contributors since the project started in 2015.

Taking over the project will help keep it up-to-date and allow us to bring additional features to it in a more timely manner. This release brings the Jenkins plugin to feature parity with our other [build server plugins]() including support for build information and automating release note generation. 

### Build Information

### Release Notes

Shipping  

## Breaking Changes

TODO: 

## Upgrading

As usual the [steps for upgrading Octopus Deploy](https://octopus.com/docs/administration/upgrading) apply. Please see the [release notes](https://octopus.com/downloads/compare?to=2019.11.0) for further information. Self-Hosted Octopus customers can [download](https://octopus.com/downloads/2019.11.0) the latest release now. For Octopus Cloud, you will start receiving the latest bits next week during your maintenance window. 

## Wrap up

That’s it for this release. We're thrilled that our suite of supported build server plugins now includes: 

- Jenkins
- JetBrains TeamCity
- Microsoft Azure DevOps
- Bamboo

 Feel free to leave us a comment, and let us know what you think! Happy deployments!
