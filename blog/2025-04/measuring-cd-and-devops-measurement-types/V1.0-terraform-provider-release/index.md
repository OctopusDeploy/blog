
--- 
title: V1.0 Release of Octopus Deploy Terraform Provider 
description: Find out all the changes we've made for the V1.0 release of the Terraform Provider
Octopus author: venkatesh.vasudevan@octopus.com 
visibility: public 
published: 2025-05-05  
metaImage: img-terraform.png 
bannerImage: img-terraform.png 
tags:    
- Configuration as Code   
- Terraform
- Product
---

!toc 

## Introduction

The Octopus Deploy Terraform provider has officially reached version 1.0, marking a significant milestone in its development. This release introduces expanded capabilities, fixes long-standing bugs, and provides improved documentation to enhance user experience. Below, we’ll explore the key updates and features included in this release.

### Expanded capabilities
Version 1.0 brings substantial improvements to managing Octopus Deploy resources through Terraform. Earlier versions of the
provider lacked support for creating all resource types available in the Octopus UI. This limitation has been addressed, enabling comprehensive resource management directly from Terraform. While most resources are now supported, a few exclusions remain—these are intentional and thoroughly documented for transparency.

### Enhanced documentation with practical examples
To ensure a smooth user experience, the updated documentation includes detailed examples and best practices for using the Terraform provider effectively. Five key scenarios have been outlined to guide users. On top of this the documentation found at [the Terraform registry](https://registry.terraform.io/providers/OctopusDeployLabs/octopusdeploy/latest/docs) now has examples for all datasources and resources.


### Bug Fixes and Provider improvements
Several long-standing bugs have been resolved in this release, ensuring a more stable and reliable experience as well as improvements to the provider:
- **New Deployment Process Resource:** We’ve made it easier to define and manage deployment processes by making each step its own resource as well as introducing a step order resource for users to easily manage deployment process step order. You can find out more info on how it works at the official docs 
- **Migration to Terraform Framework:** Transitioning from the Terraform SDK to the Framework has improved safety, reliability, and adaptability for future updates.
- **Adoption of Semantic Versioning (SemVer):** The provider now adheres strictly to SemVer principles, helping users manage version changes more predictably.

### Migration Guidance
As this is a major version release, some breaking changes may require updates to existing configurations. To assist with this transition, a comprehensive migration guide is available. Users are encouraged to review it carefully before upgrading their environments

### Octo-Terra Tool
As we introduce a new deployment process resource, we’ve improved our [Octo-terra](https://octopus.com/docs/administration/migrate-spaces-with-octoterra) tool to enable export of existing deployment processes in the new resource format.

### Provider migrated to Octopus Deploy repository 
The Terraform provider has always sat in our OctopusDeployLabs repository, which signalled that it was an experimental integration. We’ve decided to move it to our official repository, elevating it to its first-class integration status.


### An important note on Config-as-Code Projects and the Terraform Provider
We’ve observed customers using both Terraform and Config-as-Code together to manage their projects and runbooks. While both are powerful tools, combining them has led to unexpected behaviours which can break instances.
To ensure stability, we’re taking a clear stance: from version 1.0 onwards, you can manage your Octopus projects or runbooks using *either* Config-as-Code or Terraform—but not both.
If you attempt to manage a Config as Code project or runbook with Terraform, the provider will return an error. This change is designed to protect your instance and ensure predictable behavior when managing your Octopus instance.

### Looking ahead

The v1.0 release solidifies the Octopus Deploy Terraform provider as a robust tool for managing your Octopus instance as code. By simplifying resource management and improving documentation, this update empowers teams to work more efficiently with the Terraform provider

For detailed information on all changes and enhancements, refer to the official documentation or visit the Terraform registry page for the provider.
