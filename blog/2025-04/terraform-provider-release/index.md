--- 
title: "V1.0 release of the Octopus Deploy Terraform Provider" 
description: Find out all the changes we've made for the V1.0 release of the Terraform Provider.
author: venkatesh.vasudevan@octopus.com 
visibility: public 
published: 2025-05-05-1400  
metaImage: 
bannerImage:  
bannerImageAlt: Terraform branded crane lifting an Octopus branded box.
isFeatured: false
tags: 
- Product   
- Configuration as Code   
- Terraform
---

The Octopus Deploy Terraform provider has officially reached version 1.0. This release introduces expanded capabilities, fixes long-standing bugs, and provides improved documentation. 

In this post, I take you through the key updates and features included in this release.

## Expanded Terraform Provider capabilities

Version 1.0 brings substantial improvements to managing Octopus Deploy resources through Terraform. Earlier versions of the provider lacked support for creating all resource types available in the Octopus UI. We've addressed this limitation, enabling comprehensive resource management directly from Terraform. While we support most resources now, there are still a few exclusions. These are intentional and thoroughly documented for transparency.

## Clearer documentation with practical examples

For a better user experience, we updated our documentation to include detailed examples and best practices for using the Terraform Provider effectively. We outlined 5 key scenarios to guide you. The documentation found at [the Terraform registry](https://registry.terraform.io/providers/OctopusDeployLabs/octopusdeploy/latest/docs) now also has examples for all data sources and resources.


## Bug fixes and improvements

We resolved several long-standing bugs in this release. This improves the provider and ensures a more stable and reliable experience.

- New deployment process resource: We made it easier to define and manage deployment processes by making each step its own resource. We also introduced a step order resource so you can easily manage the deployment process step order. You can find out more info on how it works in the official docs.
- Migration to Terraform framework: Transitioning from the Terraform SDK to the framework has improved safety, reliability, and adaptability for future updates.
- Adoption of semantic versioning (SemVer): The provider now adheres strictly to SemVer principles. This helps you manage version changes more predictably.

## Migration guidance

As this is a major version release, some breaking changes may require updates to existing configurations. To help with this transition, we have a comprehensive migration guide. Please review it carefully before upgrading your environments.

### Octo-Terra tool

As we introduced a new deployment process resource, we improved our [Octo-terra](https://octopus.com/docs/administration/migrate-spaces-with-octoterra) tool to enable export of existing deployment processes in the new resource format.

### Provider migrated to Octopus Deploy repository 

The Terraform provider has always lived in our OctopusDeployLabs repository, which signalled that it was an experimental integration. We decided to move it to our official repository, elevating it as a core integration.


## Configuration as Code projects and the Terraform Provider

We observed customers using Terraform and Config as Code together to manage their projects and runbooks. While they're both powerful tools, combining them led to unexpected behaviours that can break instances.

To ensure stability, we’re taking a clear stance: from version 1.0 onwards, you can manage your Octopus projects or runbooks using *either* Config as Code or Terraform—but not both.

If you try to manage a Config as Code project or runbook with Terraform, the provider will return an error. We designed this change to protect your instance and ensure predictable behavior when managing your Octopus instance.

## Conclusion

The v1.0 release solidifies the Octopus Deploy Terraform provider as a robust tool for managing your Octopus instance as code. By simplifying resource management and improving documentation, teams can work more efficiently with the Terraform provider

For detailed information on all changes and enhancements, please read the docs or visit the Terraform registry page for the provider.

Happy deployments!