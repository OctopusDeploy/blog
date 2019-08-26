---
title: What is CNAB
description: A look at the Cloud Native Application Bundle (CNAB) specification, what problems it solves, and the pros and cons of the tooling.
author: matthew.casperson@octopus.com
visibility: private
published:
metaImage:
bannerImage:
tags:
 - Octopus
---

Working with cloud infrastructure can be a daunting task. Each cloud provider maintains their own CLI tools and preferred deployment strategies, cross platform tools like Terraform, Ansible, Puppet and Chef take a significant investment to learn, and once you have spun up the base infrastructure you may then be faced with managing yet more deployments to platforms like Docker or Kubernetes.

It is safe to say that these days cloud deployments will almost always require multiple tools and credentials, which is a challenge to maintain. The Cloud Native Application Bundle (CNAB) specification is a response to the growing complexity of deploying and managing cloud infrastructure.

In this post we'll take a look at the kind of problems CNAB solves, the tools currently available to work with CNAB, and the pros and cons of using CNAB today.

## What Problem does CNAB Solve?

To understand the problem CNAB solves, we can look at a problem we have faced at Octopus with the [Terraform provider](https://github.com/OctopusDeploy/terraform-provider-octopusdeploy).

The aim of this provider is to allow an Octopus server to be configured via Terraform, and the functionality is exposed as a custom Terraform plugin. However, to use the plugin you must manually download it, place it in the correct directory and reference it with a command line argument passed to Terraform. While there is [much discussion](https://github.com/hashicorp/terraform/issues/15252) in the Terraform community about the best way to distribute plugins, it remains a manual process for now.

Because it is a manual process, the burden of providing the plugin and documenting its use falls on us. For cross platform tools like Terraform, this is not an insignificant burden, as it means that the process is ideally documented for Windows, MacOS and Linux. Wouldn't it be nice if we could bundle all the required tools and scripts into a single, self container deployable artifact?

This is exactly the kind of scenario that CNAB was designed for. A CNAB bundle is essentially a collection of Docker images containing everything required to perform an installation against a remote resource. In this example, the CNAB bundle would contain the Terraform executable, the plugin, the Terraform templates and the scripts required to execute everything. The resulting bundle removes the need for the end user to download and configure individual tools, and instead gives them a self contained installer.

In providing a self contained installation bundle, CNAB addresses a number of common issues that arise when automating complex deployments with features like:

* Credential management
* Offline installations
* Versioned installers
* Signed and verifiable installers
* Audit trails
* Uninstallation
* Point and click installers

## How do you use CNAB?

CNAB itself is only a specification, and it is up to providers to implement it. The [Duffle](https://github.com/deislabs/duffle) project provides a reference implementation of the CNAB specification, and we'll use this tool throughout the rest of the blog.

The Duffle executable can be downloaded from the project's [GitHub releases](https://github.com/deislabs/duffle/releases) page. Precompiled binaries are provided for Windows, Linux and MacOS.

![](demo.svg "width=500")
