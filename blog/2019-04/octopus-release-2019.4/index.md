---
title: Automatic Release Notes and Work Item Tracking - Octopus Deploy 2019.4
description: Octopus 2019.4 introduces automatic release notes generation and rich build and work item tracking.
author: rob.pearson@octopus.com
visibility: public
published: 2019-04-08
metaImage: blogimage-shipping-2019-4.png
bannerImage: blogimage-shipping-2019-4.png
tags:
 - New Releases
 - Jira
 - Work Items
 - Release Notes
---

We are shipping Octopus Deploy 2019.4 with a focus on enhancing the feedback loop in your CI/CD pipeline.  If you have ever wanted better traceability of your requirements through build and deployment, this update is for you. 

This release includes some great updates: 

* [Build information and work item tracking](../metadata-and-work-items/index.md) captures metadata about work items and build details and surfaces them with links to the appropriate sites.
* [Release notes templates](../release-notes-templates/index.md) and automatic release notes generation makes it easy to generate and share the changes that are going into your releases.
* [Octopus integration with Jira](../../2019-05/octopus-jira-integration/index.md) which provides two-way links between your Jira issues and your deployments.

## Integrations

To take full advantage of the features in this release, you will need to update your Octopus Deploy build server plugins and install the Octopus Deploy Jira plugin if you use it. 

* [JetBrains TeamCity plugin](https://octopus.com/downloads/2019.4.0)
* [Atlassian Bamboo Plugin](https://marketplace.atlassian.com/apps/1217235/octopus-deploy-bamboo-add-on)
* [Atlassian Jira Plugin](https://marketplace.atlassian.com/apps/1220376/octopus-deploy-for-jira)

NOTE: We're also updating our Azure DevOps web extension and it should be available soon.

## Breaking Changes

There are some very slight changes to the format of the output returned by the `Octopus.Server.exe` `show-configuration` command. This is unlikely to affect you, but if you are using this to drive automation, please test the new release before upgrading.

## Upgrading

As usual, please follow the [normal steps for upgrading Octopus Deploy](https://octopus.com/docs/administration/upgrading). Please see the [release notes](https://octopus.com/downloads/compare?to=2019.4.0) for further information.

* Self-hosted Octopus customers can start using these features today by installing [Octopus Server 2019.4](https://octopus.com/downloads). Note `2019.4` is a fast lane release without [long-term support](https://octopus.com/docs/administration/upgrading/long-term-support). This featureset will be included in the next [LTS](https://octopus.com/docs/administration/upgrading/long-term-support) release of Octopus at the end of Q2 2019.

* Octopus Cloud customers will start receiving the latest bits in about 2 weeks during their maintenance window.

## Wrap up

That's it for this month. Feel free to leave us a comment and let us know what you think! Happy deployments!
