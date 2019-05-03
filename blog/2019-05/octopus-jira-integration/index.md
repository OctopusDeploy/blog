---
title: From Idea to Production - Octopus plugin for Alassian Jira Software Cloud
description: Our new Octopus Deploy plugin for Jira makes it easy to plan, track and ship software with end-to-end visibility of software pipeline.
author: rob.pearson@octopus.com
visibility: public
published: 2019-05-06
metaImage: 
bannerImage: 
tags:
 - New Releases
 - Jira
 - Work Items
 - Release Notes
---

<iframe width="560" height="315" src="https://www.youtube.com/embed/TODO" frameborder="0" allowfullscreen></iframe>

We recently shipped our [Octopus Deploy plugin for Jira](https://marketplace.atlassian.com/apps/1220376/octopus-deploy-for-jira), and I thought I'd explore it a bit as it unlocks some very useful scenarios. 

Building great software often requires using multiple tools and services, but finding the right ones and getting them to talk to each other can be a headache. Atlassian's [Jira](https://atlassian.com/jira) is a popular application that helps teams to plan, track, and release great software whereas Octopus Deploy helps teams automate their development and operations processes in a fast, repeatable and reliable manner. Together, they enable teams to get better visibility and traceability into their software pipeline from idea to production.

Integrating Octopus and Jira unlocks three key scenarios: 

* **[See when features or bug fixes are deployed to Prod](/blog/2019-05/octopus-jira-integration/index.md#See-when-features-or-bug-fixes-are-deployed-to-Prod).** 'Done' means deployed to production and this is now visible directly in your Jira issues. See when your team finishes a new feature or bug fix and deploys it to production. 
* **[See the Jira issues included in Octopus releases](/blog/2019-05/octopus-jira-integration/index.md#See-the-Jira-issues-included-in-Octopus-Releases).** It's now possible to see which Jira issues (work items) Octopus includes in releases with links back to Jira for further details.
* **[Generate release notes automatically](/blog/2019-05/octopus-jira-integration/index.md#Generate-and-share-Release-Notes-automatically).** Octopus can now intelligently generate release notes visible directly your deployments to your environments like test and production. Share them with your team, management or executives via email, slack and more.

Note: The Octopus Deploy plugin is only compatible with Jira Cloud as Jira Server does not support the APIs required to enable this functionality.

## See when features or bug fixes are deployed to Prod

> Done means deployed to production

If you've ever worked with development teams, you would have heard someone say it's 98% done and then it takes week for it to go live. This lead to teams saying work is only done when it's been deployed to production. It's now simple to see if a new feature or hotfix is marked done and also deployed to prod. You can click through to Octopus for further details.

![Jira issue with deployment details](jira-issue-with-deployments.png "width=500")

This enables greater visibility and insight for your team, managers and executives in the tool they're most comfortable with. 

## See the Jira issues included in Octopus Releases

Software normally runs through a CI/CD pipeline on it's way to production. Developers push code to source code repositories like GitHub, build servers, like Bamboo and TeamCity, build it and Octopus deploys it. Traditionally, the linkages between each of those steps can be lost but this is a now a thing of the past. 

Using our Jira plugin and one of our build server plugins, it's now possible to retain and see work item and build details directly in Octopus. 

![Octopus release details](octopus-release-details.png "width=500")

This allows teams to see the build and requirements details (Jira issues) that contributed to to a release giving end-to-end tracability from issue to deployment. You can click through to Jira for more information.

## Generate and share Release Notes automatically

Using multiple tools can make it hard to track which features are included in which release. It's hard to find which commits and builds contributed to release and deployments. Project managers and release notes make this much easier to find and understand but it's generally a manual process. Integrating Octopus and Jira enables this process to be fully automated. Release notes are calculated automatically when deploying. Octopus knows which issues have already been deployed to an environment so it can easily generate release notes showing what's new in test or production environments. 

![Octopus release notes](octopus-release-notes.png "width=500")

Reading releases in Octopus is useful but sharing them via email, slack or other mediums is even better. Octopus makes it easy to send release notes to your project management group or leadership team after every successful deployment to production keeping everyone in the loop. 

## Wrap up

In summary, [Jira](https://atlassian.com/jira) and [Octopus Deploy](https://octopus.com) work together to give you better end-to-end visibility of your software pipeline.

If your already using Jira and Octopus Deploy, I highly recommend downloading the [Octopus Deploy plugin for Jira](https://marketplace.atlassian.com/apps/1220376/octopus-deploy-for-jira) and trying this out. Read our [docs](https://octopus.com/docs/api-and-integration/metadata/jira) about how to connect the two applications and that's it. 

I also recommend looking at our build server plugins so you can take full advantage of the features above. 

* [Atlassian Bamboo plugin](https://marketplace.atlassian.com/apps/1217235/octopus-deploy-bamboo-add-on)
* [Jenkins plugin](https://plugins.jenkins.io/octopusdeploy) - _Full support for Jira issues is coming soon!_
* [Jetbrains TeamCity plugin](https://plugins.jetbrains.com/plugin/9038-octopus-deploy-integration)
* [Microsoft Azure DevOps extension]() - _Full support for Jira issues is coming soon!_

If your team is using either Jira or Octopus Deploy, you can try the other service for free to see how it works for you. 

[Try Octopus Deploy for free](https://octopus.com/trial)
[Try Jira Software Cloud for free](https://www.atlassian.com/software/jira/try)

Happy deployments!