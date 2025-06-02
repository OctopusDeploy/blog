--- 
title: V1.0 release of the Octopus Deploy Terraform Provider
description: Find out all the changes we made for the V1.0 release of the Terraform Provider.
author: venkatesh.vasudevan@octopus.com
visibility: public
published: 2025-06-04-1400
metaImage: img-terraform-provider-2025.png
bannerImage: img-terraform-provider-2025.png
bannerImageAlt: Terraform branded crane lifting an Octopus branded box.
isFeatured: false
tags:
  - Product
  - Terraform
  - Configuration as Code
---

The Octopus Deploy Terraform Provider has officially reached version 1.0. This release introduces expanded capabilities, fixes long-standing bugs, and provides improved documentation. 

In this post, I take you through the key updates and features included in this release.

## Expanded Terraform Provider capabilities

Version 1.0 brings substantial improvements to managing Octopus Deploy resources through Terraform. Earlier versions of the Provider lacked support for creating all resource types available in the Octopus UI. We addressed this limitation, enabling comprehensive resource management directly from Terraform. While we support most resources now, there are still a few exclusions. These are intentional and thoroughly [documented](https://registry.terraform.io/providers/OctopusDeploy/octopusdeploy/latest/docs) for transparency.

## Clearer documentation with practical examples

For a better user experience, we updated the [documentation] (https://registry.terraform.io/providers/OctopusDeploy/octopusdeploy/latest/docs) to include detailed examples and best practices for using the Terraform Provider effectively. We outlined 5 key scenarios to guide you. The documentation found at [the Terraform registry](https://registry.terraform.io/providers/OctopusDeploy/octopusdeploy/latest/docs) now also has examples for all data sources and resources.


## Bug fixes and improvements

We resolved several long-standing bugs in this release. This improves the Provider and ensures a more stable and reliable experience.

- New deployment process resource: We made it easier to define and manage deployment processes by making each step its own resource. We also introduced a step order resource so you can easily manage the deployment process step order. You can find out more info on how it works in the [official docs](https://registry.terraform.io/providers/OctopusDeploy/octopusdeploy/latest/docs).
- Migration to Terraform framework: Transitioning from the Terraform SDK to the framework has improved safety, reliability, and adaptability for future updates.
- Adoption of semantic versioning (SemVer): The Provider now adheres strictly to SemVer principles. This helps you manage version changes more predictably.

## Migration guidance

As this is a major version release, some breaking changes may require updates to existing configurations. To help with this transition, we have a [comprehensive migration guide](https://registry.terraform.io/providers/OctopusDeploy/octopusdeploy/latest/docs). Please review it carefully before upgrading your environments.


### Provider migrated to Octopus Deploy repository 

The Terraform Provider has always lived in our OctopusDeployLabs repository, which signalled that it was an experimental integration. We decided to move it to our [official repository](https://github.com/octopusdeploy), elevating it as a core integration.


## Version-controlled projects and the Terraform Provider

We observed customers using Terraform and version control together to manage their projects and runbooks. While they're both powerful tools, combining them led to unexpected behaviours that can break instances.

To ensure stability, we’re taking a clear stance: from version 1.0 onwards, you can manage your Octopus projects or runbooks using *either* version control or Terraform—but not both.

If you try to manage a version-controlled project or runbook with Terraform, the Provider will return an error. We designed this change to protect your instance and ensure predictable behavior when managing your Octopus instance.

## Conclusion

The v1.0 release solidifies the Octopus Deploy Terraform Provider as a robust tool for managing your Octopus instance as code. By simplifying resource management and improving documentation, teams can work more efficiently with the Terraform provider

For detailed information on all changes and enhancements, please [read the docs](https://registry.terraform.io/providers/OctopusDeploy/octopusdeploy/latest/docs) or visit the Terraform registry page for the Provider.

Happy deployments!
