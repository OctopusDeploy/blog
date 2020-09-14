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

I'm thrilled to share that we've shipped Octopus 2020.4 with some improvements to some existing features that make them a lot more useful.

* **Structured variable replacement** is an update to our JSON configuration variable replacement support to make it more useful. It now supports JSON, YAML, XML and Properties files. This is a huge improvement for numerous platforms but it's especially valuable for Java teams. This update unlocks end-to-end Java pipelines with Octopus and 

* **Octopus Cloud Linux Workers** unlocks better cross-platform support with 

Octopus Cloud provides dynamic workers to execute scripts against your services and infrastructure. We are building better cross-platform support by adding images for Windows 2019 and Ubuntu 20.04. All worker images will support Execution Containers thus providing the ability to simplify dependency management and streamline automation tooling.

[End-to-end Java pipelines with Jenkins and Octopus] -  
[Structured variable replacement] - New addition is structured variable replacement with support for YAML, XML and Property files.
[Simpler DevOps dependency and tool management] with Execution Containers and Workers (Win and Linux)

* [Title](blog/2020-09/octopus-release-2020-4/index.md#title): Description
* [Title](blog/2020-09/octopus-release-2020-4/index.md#title): Description

This release is the [fourth of six in 2020](/blog/2020-03/releases-and-lts/index.md), and it includes six months of long-term support. The following table shows our current versions with long-term support:

| Release               | Long-term support  | LTS end date |
| --------------------- | ------------------ | ------------ |
| Octopus 2020.4        | Yes                | TODO         |
| Octopus 2020.3        | Yes                | 2021-01-20   |
| Octopus 2020.2        | Yes                | 2020-09-31   |
| Octopus 2020.1        | Expired            | 2020-08-24   |

Keep reading to learn more about the updates.

## Structured variable replacement

TODO

## Octopus Cloud Windows and Linux Workers

TODO

## Breaking changes

This release includes 



## Upgrading

Octopus Cloud users are already running this release, and self-hosted Octopus customers can [download](https://octopus.com/downloads/2020.3.0) the latest version now.  

As usual, the [steps for upgrading Octopus Deploy](https://octopus.com/docs/administration/upgrading) apply. Please see the [release notes](https://octopus.com/downloads/compare?to=2020.3.0) for further information.

## What’s coming in Octopus 2020.5?

Check out our [public roadmap](https://octopus.com/roadmap) to see what’s coming next and register for updates. That said, we're planning to launch our Config as Code early access preview (EAP) so stay tuned for more information on that. 

## Conclusion

Octopus 2020.4 is now generally available, and it includes improvements TODO. We hope you enjoy it! 

Feel free to leave a comment, and let us know what you think! Happy deployments!

## Related posts

* [](/blog/2020-07/using-jenkins-pipelines/index.md)
