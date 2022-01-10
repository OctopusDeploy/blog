---
title: Octopus Deploy 2019.10 - Flexible Linux deployments, PowerShell Core support, Operations Runbooks EAP
description: Octopus 2019.10 introduces Tentacle for Linux for flexible Linux deployments, built-in PowerShell Core support, simpler build information and releas notes, and early access to Runbooks for your operations teams.
author: rob.pearson@octopus.com
visibility: public
published: 2019-10-16
metaImage: octopus-2019.10-release-image.png
bannerImage: octopus-2019.10-release-image.png
tags:
 - Product
---

<iframe width="560" height="315" src="https://www.youtube.com/embed/pBW_gypiS6w" frameborder="0" allowfullscreen></iframe>

We’re excited to launch **Octopus Deploy 2019.10**. This is a feature packed release with some great benefits. Headlining the release is Tentacle for Linux. This much requested feature provides greater flexibility for teams deploying to Linux and unlocks the ability to deploy to highly secured servers without opening port 22. Tentacle for Linux and Linux deployments over SSH combined with scripting in Bash, PowerShell Core, or Python, and a first-class step to deploy your web apps and services to NGINX, make Octopus a great option for Linux deployments. 

<h2>In this post</h2>

!toc

## Tentacle for Linux

![Tentacle for Linux configuration](linux-tentacle.png "width=600")

Octopus first introduced support for Linux deployments over secure shell (SSH) in version 3.0 and it’s a popular option for deployments. However, some companies operate in highly secure environments where it’s not possible to open port 22 on production servers, for instance, when web applications operate in their own DMZs with no incoming connections permitted other than HTTPS for web traffic. 

Thankfully, this scenario is now possible with Tentacle for Linux. Our Tentacle agent is a lightweight service that enables secure communication between the Octopus Server and deployment targets in both listening and polling modes. In polling mode, the targets themselves phone home and contact the Octopus Server to see if there is any deployment work to do and if there is, they execute it. 

[Learn more](https://octopus.com/docs/infrastructure/deployment-targets/linux/tentacle)

## PowerShell Core support

![Powershell core support](powershell-core.png "width=600")

This release adds built-in PowerShell Core support enabling teams to write cross-platform scripts using Microsoft’s actively maintained automation framework. Our PowerShell Core support just works, and it pairs very well with our expanded support for Linux deployments. Many development and operations teams are comfortable writing PowerShell scripts, and PowerShell Core enables people with PowerShell skills to write rich scripts for Linux platforms.

**Windows platforms**

Octopus will use Windows PowerShell by default and provides the option to use PowerShell Core if desired. You can customize this by configuring the PowerShell edition for steps that support custom scripts.

**Linux platforms**

Octopus will automatically execute scripts with PowerShell Core if it’s installed. 

[Learn More](http://octopus.com/docs/deployment-examples/custom-scripts/powershell-core)

## Improved build information and work item tracking

![Octopus build information repository](build-information-repository.png  "width=600")

Octopus 2019.4 introduced build information and work item tracking. This is a popular feature, but we received feedback the package metadata functionality was hard to find because it was buried under the package details in the built-in package repository and tricky to understand. As a result, we’ve promoted this functionality to a top-level feature called **Build Information** within the Octopus Library making it more accessible and much easier to understand. We’ve also updated our suite of build server plugins to reflect the name change.

This feature-set gives teams better end-to-end visibility into their CI/CD pipelines and unlocks quick access to build and commit details. This is visible in a number of ways:

- Release notes
- Deployment changes

We previously shipped support to customize release notes templates in project settings. In this release, we’re introducing deployment change templates so you can get the same control over the structure of your deployment changes. This gives teams the ability to customize the display to suit their needs. 

[Learn more](https://octopus.com/docs/packaging-applications/build-servers/build-information)

## Simpler package-based step templates

![Custom step template example with a package parameter](step-template-package-params.png  "width=600")

Step templates are a popular way for teams to create reusable steps for use across multiple projects. We’ve made a small but significant update to make it easier to share package-based step templates across projects with the introduction of package parameters. Previously, you needed to create parameters to expose package-based properties. With package parameters, this is no longer necessary as we now automatically show our package selector when the step template is added to a project. This makes it easier to create package-based step templates, and lets teams bind against these parameters in a much simpler way.

[Learn more](https://octopus.com/docs/deployment-process/steps/custom-step-templates)

## Introducing Runbooks for Operations teams

![Octopus Operations Runbooks](runbook-details.png  "width=600")

This release also includes early access to our new Operations Runbooks feature. Until now, Octopus has been a deployment automation tool, enabling teams to deploy new releases of their software, but once the software is deployed, there are many other processes that teams need to automate. These can include file cleanup, nightly backups, disaster recovery, and other scripts and manual processes. With Runbooks, teams can use Octopus to automate everything involved in keeping modern software running in production.

We’d love feedback on this feature so join the discussion on our [community slack](https://octopus.com/slack) in the `#runbooks` channel. Our docs cover all the details on how to get started.

[Learn more](/blog/2019-10/introducing-operations-runbooks/index.md)

## Breaking Changes

This release includes two minor breaking changes. 

**PowerShell Core**

PowerShell Core support has introduced a change where running PowerShell scripts against non-Windows targets no longer silently ignores `.ps1` files. Octopus will now attempt to execute `.ps1` files on deployment targets. If PowerShell Core is installed on those targets, this may change the behaviour of your deployments. If PowerShell Core is not installed on those targets, the deployment will fail.

**Build Information** 

Our build information change involved renaming resources from the API through to the database. This is generally backwards compatible, however anyone using our GitHub issue tracker will need to ensure they upgrade their Octopus Server if they upgrade their [Azure DevOps extension](https://marketplace.visualstudio.com/items?itemName=octopusdeploy.octopus-deploy-build-release-tasks), [TeamCity plugin](https://plugins.jetbrains.com/plugin/9038-octopus-deploy-integration), or [Bamboo plugin](https://marketplace.atlassian.com/apps/1217235/octopus-deploy-bamboo-add-on).

## Upgrading

As usual the [steps for upgrading Octopus Deploy](https://octopus.com/docs/administration/upgrading) apply. Please see the [release notes](https://octopus.com/downloads/compare?to=2019.10.0) for further information. Self-Hosted Octopus customers can [download](https://octopus.com/downloads/2019.10.0) the latest release now. For Octopus Cloud, you will start receiving the latest bits next week during your maintenance window. 

## Wrap up

That’s it for this release. Feel free to leave us a comment, and let us know what you think! Happy deployments!
