---
title: "Octopus 2020.2: Execution containers for Workers"
description: "Octopus 2020.2 includes Execution Containers for Workers, better rolling deployments and code editor improvements."
author: rob.pearson@octopus.com
visibility: private
bannerImage: release-2020.2.png
metaImage: release-2020.2.png
published: 2022-05-25
tags:
- Product
---

![Octopus 2020.2: Execution containers for Workers](release-2020.2.png)

We're proud to ship Octopus 2020.2, our second release of the year, with some great updates and benefits.

* [Execution containers for Workers](blog/2020-05/octopus-release-2020-2/index.md#execution-containers-for-workers) enables you to execute deployment work in isolation without the need to manage dependencies and containers.
* [Rolling deployment run conditions](blog/2020-05/octopus-release-2020-2/index.md#rolling-deployment-run-conditions) adds support for run conditions to child steps within a rolling deployment.
* [Improved code editor with fast variable lookups](blog/2020-05/octopus-release-2020-2/index.md#improved-code-editor-with-fast-variable-lookups) unlocks the ability to quickly add Octopus variables into your custom scripts without touching the mouse.

This release is the [second of six in 2020](/blog/2020-03/releases-and-lts/index.md), and it includes 6 months of long term support. The following table shows our current releases with long term support. 

| Release               | Long term support           |
| --------------------- | --------------------------- |
| Octopus 2020.2        | Yes                         |
| Octopus 2020.1        | Yes                         |
| Octopus 2019.12       | Yes                         |
| Octopus 2019.9        | Expired                     |
| Octopus 2019.6        | Expired                     |

Keep reading to learn more about the updates.

## Execution containers for Workers

TODO: Add screenshot

Workers were introduced in Octopus 2018.x and they help teams move deployment work off the Octopus Server and onto other machines running in worker pools. Common Worker scenarios include database deployments and cloud deployments where by you can create a pool of workers for that specific purpose. 

In this release, we're adding support to execute deployment work in isolated containers on Workers that are bootstrapped with common deployment tooling. Octopus is also shipping a collection of [official Docker container images] with the latest tools so make it easy to get started and you can start using them straight away. 



[Learn more](https://octopus.com/docs/deployment-process/execution-containers-for-workers)

## Rolling deployment run conditions

TODO: Add screenshot

It is now possible to add run conditions to rolling deployments. This adds great flexibility and 

[Learn more](https://octopus.com/docs/deployment-process/conditions)

## Improved code editor with fast variable lookups

TODO

## Other customer requested improvements

TODO

## Breaking changes

This release includes x breaking changes.



## Upgrading

Octopus Cloud users are already running this release, and self-hosted Octopus customers can [download](https://octopus.com/downloads/2020.2.0) the latest release now.  

As usual, the [steps for upgrading Octopus Deploy](https://octopus.com/docs/administration/upgrading) apply. Please see the [release notes](https://octopus.com/downloads/compare?to=2020.2.0) for further information.

## What's coming in Octopus 2020.3?

Check out our [public roadmap](https://octopus.com/roadmap) to see what's coming next. We're about to start work on some incredible new features, including deep git integration, unlocking pipeline as code.

## Conclusion

TODO: Write conclusion

Feel free to leave us a comment, and let us know what you think! Happy deployments!

## Related posts

- Related blog post 1
- Related blog post 2
