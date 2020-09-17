---
title: "Octopus 2020.4: TODO"
description: "Octopus 2020.4 includes "
author: rob.pearson@octopus.com
visibility: private
bannerImage: release-2020.4.png
metaImage: release-2020.4.png
published: 2021-09-07
tags:
- Product
---

<iframe width="560" height="315" src="https://www.youtube.com/embed/xJqjn4s2VCI" frameborder="0" allowfullscreen></iframe>

I'm thrilled to share that we've shipped Octopus 2020.4. This release bring together a couple of features that have been in development for a while. Together, they make it easier for teams to deploy and maintain web applications and services written in Java, NodeJS, Python and Ruby and more.

//Lee: These titles are boring as a sack of hammers. I need something to spice up Better configuration variable replacement. I used this instead of the featuer name (Structured variable replacement) to communicate the value/benefit but it doesn't feel right. Any suggestions.

* **[Better configuration file updates with structure variable replacement](/blog/2020-09/octopus-release-2020-4/index.md#variables)**: is an update to our JSON configuration variable replacement support to make far more useful. It now supports JSON, YAML, XML and Java Properties files. This is a huge improvement for numerous platforms but it's especially valuable for Java teams.

* **[Octopus Cloud: Cross-platform Worker Pools and simpler dependency management](/blog/2020-09/octopus-release-2020-4/index.md#cross-platform-dynamic-workers)**. Octopus Cloud provides dynamic workers to execute scripts against your services and infrastructure. This update adds better cross-platform support with images for Windows 2019 and Ubuntu 18.04. All Worker support execution containers which let you execute deployment work in isolation without the need to manage dependencies and containers.

This release is the [fourth of six in 2020](/blog/2020-03/releases-and-lts/index.md), and it includes six months of long-term support. The following table shows our current versions with long-term support:

| Release               | Long-term support  | LTS end date |
| --------------------- | ------------------ | ------------ |
| Octopus 2020.4        | Yes                | TODO         |
| Octopus 2020.3        | Yes                | 2021-01-20   |
| Octopus 2020.2        | Yes                | 2020-09-31   |
| Octopus 2020.1        | Expired            | 2020-08-24   |

Keep reading to learn more about the updates.

## Better configuration file updates with structure variable replacement {#variables}

// TODO: Screenshot

One of Octopus' magical features is how it can automatically update your configuration files as you promote your applications to production. Historically, this supported a number of Microsoft configuration file formats, primarily XML based, as well as some general approaches like JSON support and token replacement. This was fantastic however if you weren't using XML or JSON files, you needed to insert and maintain tokens in your configuration files which can be awkward and time consuming. 

This is no longer a problem because we're introducing support for strutured variable replacement. Structured variable replacement support modern configuration file formats including:

* YAML
* JSON 
* XML 
* Java Properties files

For other configuration files formats, our token replacement support, via the Substitute variables in files feature, has you covered. 

This new feature is an evolution of our JSON configuration variable replacement feature. The benefit of this support is that it's automatic and convention based. If this featuer is turned on, Octopus will automatically update configuration settings with names that many that your project variables. The magic of this feature is that you can scope your variables to environments so deploying releases to Dev, Test, Staging and Production environments is repeatable and reliable.

This update makes it far easier to configure automated deployments for Java applications like Spring web apps and services, web apps written in Python, NodeJS services, Ruby on Rails web apps and more.

[Learn more](/blog/2020-08/spring-environment-configuration/index.md)

## Octopus Cloud: Cross-platform Worker Pools and simpler dependency management {#cross-platform-dynamic-workers}

Octopus 2020.2 includes better cross-platform support for dynamic workers including full support for Execution Containers.

:::hint
Workers enable you to shift DevOps automation work onto other machines running in pools for specific purposes like deploying to Kubernetes, cloud platforms like Azure and AWS as well as database deployments. You can create a pool of dedicated workers that can be utilized by multiple projects and teams. They're a great tool for scaling your deployments and runbooks.

See [our documentation](https://octopus.com/docs/infrastructure/workers) for more information.
:::

## Built-in Windows and Linux Worker Pools

Octopus Cloud provides dynamic workers to execute scripts against your services and infrastructure without the need to manage your own virtual machines or other compute resources. This greatly simplifies the ability to execute automation scripts, deployment or runbook related, against cloud services, database servers or Kubernetes clusters. 

With Octopus 2020.4, we have improved the cross platform support for dynamic workers by rounding out our support for modern Windows and Linux operation systems:
* Windows 2019
* Ubuntu 20.04

NOTE: Windows 2016 is still supported and this is explained futher in the learn more link below.

These are Virtual Machien iamges and they are bootstrapped with basic tooling including the following: 

* Docker
* Bash
* PowerShell Core
* Python
* Octopus CLI

If you need additional tools, you could always install them as a part of the script or build yoru own customer worker pools with your own machines. 

[Learn more](/blog/2020-09/linux-worker-pools-on-octopus-cloud/index.md)

## Execution Containes for Workers

With this update, Execution Containes for Workers is now generally avialable and we're removing the early access program feature flag. This feature enable you to execute automation work in isolated containers on workers and reduces the need to manage automation tooling and dependencies.

Previously, you needed to ensure the machines in your worker pools (including dynamic workers) had the necessary tools required for your deployments, and you needed to maintain their OS and tool versions. This approach could also be problematic if different teams required different versions of specific tools that don’t install side by side. Also, Octopus bundled some tools, but it was still a challenge to keep them up to date.



We ship and maintain a colleciton of official container images bootstrapped with common tooling however it's also possible to extend and customize these images to suit your team's needs. 

[Learn more](/blog/2020-06/execution-containers/index.md)

## Breaking changes

This release doesn't include any breaking changes.

## Upgrading

Octopus Cloud users are already running this release, and self-hosted Octopus customers can [download](https://octopus.com/downloads/2020.3.0) the latest version now.  

As usual, the [steps for upgrading Octopus Deploy](https://octopus.com/docs/administration/upgrading) apply. Please see the [release notes](https://octopus.com/downloads/compare?to=2020.3.0) for further information.

## What’s coming in Octopus 2020.5?

Check out our [public roadmap](https://octopus.com/roadmap) to see what’s coming next and register for updates. That said, we're planning to launch our Config as Code early access preview (EAP) so stay tuned for more information on that. 

## Conclusion

Octopus 2020.4 is now generally available, and it includes improved configuration file updates, better cross-platform support for Octopus Cloud including support for execution container for workers. We hope you enjoy it! 

Feel free to leave a comment, and let us know what you think! Happy deployments!

## Related posts

* [](TODO)
