---
title: Automatic Release Notes and Rich Build and Work Item Tracking - Octopus Deploy 2019.4
description: Octopus 2019.4 introduces automatic release notes generation and rich build and work item tracking.
author: rob.pearson@octopus.com
visibility: public
published: 2019-04-04
metaImage: 
bannerImage: 
tags:
 - New Releases
 - Jira
 - Work Items
 - Release Notes
---

We are shipping Octopus 2019.4 with a focus on enhancing the feedback loop in your CI/CD pipeline. If you have ever wanted better traceability of your requirements through build and deployment, this update is for you. 

This release includes some great updates: 

* [Build information and work item tracking](../metadata-and-work-items/index.md) captures metadata about work items and build details and surfaces them with links to the appropriate sites.
* [Release notes templates](../release-notes-templates/index.md) and automatic release notes generation makes it easy to generate and share the changes that are going into your releases.
* [Octopus integration with Jira](https://g.octopushq.com/JiraIssueTracker) which provides two-way links between your Jira issues and your deployments.

## Integrations

To take full advantage of the features in this release, you will need to update your Octopus Deploy build server plugins and install the Octopus Deploy Jira plugin if you use it. 

* [JetBrains TeamCity plugin](https://octopus.com/downloads/2019.4.0)
* [Atlassian Bamboo Plugin](https://marketplace.atlassian.com/apps/1217235/octopus-deploy-bamboo-add-on)
* [Atlassian Jira Plugin](https://marketplace.atlassian.com/apps/1220376/octopus-deploy-jira-plugin)

NOTE: We're working to update our Azure DevOps extension and it should be available soon.

## Breaking Changes

There are no breaking changes in this release however we've made some changes to how configuration is stored (moving some settings from the config file to the database) so we recommend taking a backup of your `OctopusServer.config` file before upgrade. 

## Upgrading

As usual, [steps for upgrading Octopus Deploy](https://octopus.com/docs/administration/upgrading) apply. Please see the [release notes](https://octopus.com/downloads/compare?to=2019.4.0) for further information.

* Self-hosted Octopus customers can start using these features today by installing [Octopus Server 2019.4](https://octopus.com/downloads). Note `2019.4` is a fast lane release without [long-term support](https://octopus.com/docs/administration/upgrading/long-term-support). This featureset will be included in the next [LTS](https://octopus.com/docs/administration/upgrading/long-term-support) release of Octopus at the end of Q2 2019.

* Octopus Cloud customers will start receiving the latest bits in about 2 weeks during their maintenance window.

## Wrap up

That's it for this month. Feel free to leave us a comment and let us know what you think! Happy deployments!