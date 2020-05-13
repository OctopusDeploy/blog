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

We're pleased to ship Octopus 2020.2, our second release of the year, with some great updates and benefits.

* [Execution containers for Workers (EAP)](blog/2020-05/octopus-release-2020-2/index.md#execution-containers-for-workers) enables you to execute deployment work in isolation without the need to manage dependencies and containers.
* [Better run conditions](blog/2020-05/octopus-release-2020-2/index.md#better-run-conditions) adds deployment process improvements including rolling deployment and machine level variable conditions.
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

// TODO: Add screenshot

Workers were introduced in Octopus 2018.7 and they help teams move deployment work off the Octopus Server and onto other machines running in worker pools. Common scenarios for Workers include database deployments and cloud deployments where by you can create a pool of workers for that specific purpose. Worker pools can also be scoped to environments to suit your needs.

In this release, we're adding support to execute deployment work in isolated containers on Workers and we're shipping a collection of official container images bootstrapped with common deployment tooling. 

Previously, you would need to ensure the servers in your worker pools (including the built-in worker pool) have the necessary tools required for your deployments and you needed to maintain the OS and tool versions. This could also be problematic if different teams required different versions of specific tools which don't install side by side. This is further complicated by the fact that Octopus bundles some tooling to simplify deployments but this is also a challenge to keep up to date. 

**Execution containers for Workers** solve these problems and more.

* **Isolated and fast execution of deployment work.** Octopus is using [Docker](https://docker.com) to execute your scripts or other deployment work in the context of a container. This is fast and efficient isolated execution.
* **Simplified dependency management with pre-built [Octopus tooling container images](https://hub.docker.com/r/octopusdeploy/worker-tools)**. There is now far less friction required to ensure you're using the right versions of the tooling that you need for your deployments. 

Our pre-built images include cross platform support for Windows 2019 and Ubuntu 18.04 and you can select the `latest` image tag or a specific version based on major, minor or specific patch verisons. We are launching with the following tools installed. 

* Powershell Core
* .NET Core SDK (3.1 LTS)
* Java SDK
* Azure CLI
* Az Powershell Core Modules
* AWS CLI
* Node.js
* kubectl
* Helm 3
* Terraform
* Python
* Azure Function Core Tools
* Google Cloud CLI
* ScriptCS (Window-only)
* F# (Windows-only)

It's also possible to build your own container images with your team's exact requirements. For example, you can build a customize image with a specific version of kubectl with the following command.

```
docker build -t my-company/worker-tools --build-arg Kubectl_Version=X.Y.Z MyDockerFile
```

[Learn more](https://octopus.com/docs/deployment-process/execution-containers-for-workers)

## Better run conditions

Run conditions allow you to custom each step in your deployment process to provide greater control over the step's execution. This release adds support for rolling deployment and machine level run conditions.

### Rolling deployment variable run conditions

// TODO: Add screenshot

It is now possible to add variable run conditions to child steps in rolling deployments. This adds greater flexibility to rolling deployments and allows you to customize the deployment process based on your specific needs. For example, you could check to see if a web service updated in a previous step is running (i.e. a a sanity test), and if so, run it re-add it back to a web farm.

[Learn more](https://octopus.com/docs/deployment-process/conditions)

### Machine-level variable run conditions

// TODO: Add screenshot

Another new addition to variable run conditions is added support for machine-level variables. The rolling deployment example above highlights this improvement as well. In such a deployment, if you check to see if a recently updated web service updated is running (i.e. a a sanity test) and set a variable to indicate the result, this is a machine-level variable that can then be resolved in a run condition in a future step within the same rolling deployment.

[Learn more](https://octopus.com/docs/deployment-process/conditions#machine-level-variable-expressions)

## Improved code editor with fast variable lookups

// TODO: Add GIF

We've also added a handy shortcut to be able to insert variables quickly without needing to click the variable lookup button. Press `Control` + `Space` on your keyboard to get a quick variable lookup menu with fuzzy search support. Select the appropriate variable using the arrow keys and then press `Enter`. This simply update is very handy once you get used to it. Try it today.

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
