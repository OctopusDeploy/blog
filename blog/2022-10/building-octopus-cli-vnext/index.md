---
title: Building Octopus CLI vNext
description: Building the next version of the Octopus CLI
author: john.bristowe@octopus.com
visibility: public
published: 2022-10-14-1400
metaImage: blogimage-building-octopus-cli-vnext.png
bannerImage: blogimage-building-octopus-cli-vnext.png
bannerImageAlt: Desktop screen in the clouds with an Octopus Deploy logo in front of it.
isFeatured: false
tags:
  - CLI
  - Product
  - Tools
---

The Octopus CLI has served us well for many years. However, it has some limitations that we wish to address. In this article, we'll share with you our plans to evolve the CLI experience with Octopus Deploy.

## The State of the Octopus CLI (octo)

The [Octopus CLI](https://github.com/OctopusDeploy/OctopusCLI) (`octo`) is built on C# and relies heavily on the [OctopusClients](https://github.com/OctopusDeploy/OctopusClients) .NET library. It provides commands to facilitate the automation of deployments and runbook execution. Notable commands include `build-information`, `create-release`, `deploy-release`, `pack`, `push`, and `run-runbook`. The Octopus CLI remains one of the most productive means for interacting with the [Octopus REST API](https://octopus.com/docs/octopus-rest-api). It empowers our customers to automate repetitive tasks while providing the flexibility to perform one-off commands. In fact, most of the integrations we build and maintain for supported platforms like Azure DevOps, GitHub Actions, and TeamCity use it to perform operations in concert with the [Octopus REST API](https://octopus.com/docs/octopus-rest-api).

The Octopus CLI has served us well for many years. However, it has some limitations that we wish to address:

- **Built for automation, not for people:** when you use the Octopus CLI, it's evident that it was built primarily as an automation tool. The CLI needs to evolve to include a human-first design.
- **Inconsistent command structure and output:** the Octopus CLI command structure is inconsistent due to commands being added over the years. The output is not governed by an overarching strategy. There are notable exceptions in the command structure that hurts consistency (e.g. pack). Furthemore, these commands are grouped by operations rather than by the target resource. This hurts the overall user experience and makes it difficult to evolve the set of commands.
- **Runtime dependencies:** despite being a self-contained executable, the reality is that the Octopus CLI has requirements of platform libraries that must be installed prior to running the binary (i.e. dependencies for .NET self-contained executables on Alpine). This is reflected in the installation script that is required for the Octopus CLI vNext for various distributions of Linux.
- **Difficult to cross-platform compile:** the build scripts are tied to Nuke and require Windows-only tools (i.e. `SignTool.exe`), which makes building cross-platform difficult ([#124](https://github.com/OctopusDeploy/OctopusCLI/issues/124))
- **No support for the Executions API:** the Octopus CLI (`octo`) does not support these operations and adding support for these operations requires a number of code paths
- **Lack of self-serve troubleshooting:** the Octopus CLI (`octo`) does not support a built-in capability for customers to troubleshoot problems such as network connectivity issues ([#220](https://github.com/OctopusDeploy/OctopusCLI/issues/220))

We wrestled with the decision of whether or not to continue building out the capabilities of the Octopus CLI and incorporate the changes we wanted to make to move things forward. After careful consideration of the different possibilities, we decided to start fresh without the constraints of 10+ years of design decisions that the Octopus CLI has baked in. This decision also mitigates the challenge of updating the Octopus CLI and exposing ourselves to downstream problems with our existing integrations. We wanted to be more opinionated and focused on customer-centric workflows that we often defer to our API.

## Introducing the New Octopus CLI (octopus)



The new Octopus CLI (`octopus`) represents an evolution of the Octopus CLI. For starters, the number of available commands will be significantly expanded. The new Octopus CLI will grow to support operations for managing resources like accounts, lifecycles, projects, and spaces. It will also feature a new capability for user interaction – we're proponents of the [Command Line Interface Guidelines](https://clig.dev/), which advocates this capability. This feature is designed to guide users through a series of questions in order to perform the operation they want in the easiest way possible:

![Demo: Create Release with Octopus CLI vNext](demo-create-release.gif)

An interactive CLI will guide both first-time and experienced users down a path that will provide the outcome they want. That stated, the new Octopus CLI will support automation as its primary use case.

![Demo: Deploy Release with Octopus CLI vNext](demo-release-deploy.gif)

The new Octopus CLI is based on the Go programming language. The language and its supporting libraries are ideally suited for building a CLI. Furthermore, we wish to use this opportunity to improve the [Go API Client for Octopus Deploy](https://github.com/OctopusDeploy/go-octopusdeploy), which underpins the [Terraform Provider for Octopus Deploy](https://github.com/OctopusDeployLabs/terraform-provider-octopusdeploy).

## Why Use Go to Build a CLI?

Go is a highly-concurrent language that is well-suited for building a CLI. Furthermore, the [Go API Client for Octopus Deploy](https://github.com/OctopusDeploy/go-octopusdeploy) has been built to support the [Terraform Provider for Octopus Deploy](https://github.com/OctopusDeployLabs/terraform-provider-octopusdeploy). It has been put through its paces. Finally, Go lends itself to a small runtime footprint through multiplatform support that's based on C++. This provides a small executable file size and a small set of requirements on the target environment – this combines to support scenarios where customers wish to use the CLI with Bash (for example) via `curl`.

## What's the Future of the Octopus CLI (octo)?

We believe customers should use whatever techniques and/or tools makes them happiest and most productive when using Octopus Deploy. If customers want to automate their workflows through the Octopus REST API or the Octopus CLI (`octo`) through integrations then that's great news. We have no interest in deprecating the Octopus CLI (`octo`) in the short-to-medium term. For the long-term, we hope to convince customers that the new Octopus CLI (`octopus`) will be compelling enough to consider a switch.

The new Octopus CLI (`octopus`) is more opinionated and intended to help simplify customer workflows from the command line. It is designed to be small, fast, and more feature-rich. It is not intended to be an exact replacement for the Octopus CLI (`octo`) and likely never will be, but our hope is that the vast majority of Octopus Deploy customers who use the Octopus CLI (`octo`) will find more value in using the new Octopus CLI (`octopus`) as we continue to improve it.

## We Need Your Help

We want to build a CLI that customers love to use. This is where we need your help! We want to hear the scenarios you would like for us to support. You can track our progress building the CLI by following the [cli repository on GitHub](https://github.com/OctopusDeploy/cli). Please feel free to watch or star the repository for updates. We have also started publishing distributables to popular package repositories like Chocolatey and Homebrew.

Happy deployments!
