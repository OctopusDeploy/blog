---
title: "Getting started with the Terraform provider for Octopus Deploy"
description: Introducing the official Terraform provider for Octopus Deploy. Learn how to use it to manage Octopus instances with Terraform.
author: michael.levan@octopus.com
visibility: public
published: 2011-01-25
metaImage: blogimage-terraform-provider-2021.png
bannerImage: blogimage-terraform-provider-2021.png
tags:
 - DevOps
 - Terraform
---

![Octopus Deploy Terraform provider: Getting started with the Beta](blogimage-terraform-provider-2021.png)

Infrastructure as code ([IaC](https://searchitoperations.techtarget.com/definition/Infrastructure-as-Code-IAC#:~:text=Infrastructure%20as%20code%2C%20also%20referred,hardware%20devices%20and%20operating%20systems.)) allows teams to create infrastructure resources (i.e. virtual machines, storage, network resources etc) without walking through wizards or numerous mouse clicks. [Hashicorp's](https://www.hashicorp.com) [Terraform](https://www.terraform.io) is one of the most popular infrastructure as code solutions. Terraform is an open-source IaC solution that allows you to define your infrastructure using a functional-based programming language called [Hashicorp Configuration Language (HCL)](https://github.com/hashicorp/hcl). 

Octopus is proud to introduce our official [Terraform provider](https://github.com/OctopusDeploy/terraform-provider-octopusdeploy). This project started as a [community initiative](https://github.com/MattHodge/terraform-provider-octopusdeploy) by [Matthew Hodgkins](https://github.com/MattHodge/) who built it to suit his needs. Down the track, [Mark Henderson](https://github.com/mhenderson-so) contributed to the project for the needs of StackExchange. We’re indebted to Matt and Mark for their efforts since the project started in 2018. Making this an official supported provider brings significant advantages, as we can keep the plugin up-to-date, improve it's testing and add new features.

In this blog post, I'll introduce the Terraform provider for Octopus Deploy, and share practical examples of how to get started with it. 

## Why Infrastructure as Code?

Automation and DevOps and are the future and we cannot continue to create infrastructure manually. It's slow and prone to errors. Humans make mistakes and it's far too easy to break systems with something as small as a typo.

Infrastructure as Code enables to you declaratively describe your infrastructure and the associated tooling allows you to test it before applying it to production systems. Storing your configuration in source control enables you to improve quality further with collaboration, code reviews and it gives you a full history of changes. 

## What is a Terraform Provider? 

Terraform is an open source tool that enables teams to provision and update infrastructure using a declarative configuration files following an infrastructure as code approach. You describe the state of the resources in configuration files and Terraform applies this to your infrascture. It can create, destroy and even detect drift between the desiged state and the actual infrastructure. 

Terraform providers are plugins which allow Terraform to interact with a wide variety of technologies. The Octopus Deploy Terraform provider allows you to interact with an Octopus instance and manage it's configuration.

## Getting started with the Terraform provider for Octopus Deploy

Our Terraform provider allows teams to provision and configure Octopus instances. This applies to both on-prem Octopus servers as well as [Octopus Cloud](https://octopus.com/pricing/cloud) instances. 

:::hint
We use this Terraform provider internally to provision and configure Octopus instances including Octopus Cloud. It's a fantastic tool and we're excited to see it used in teams in the community.
:::

This provider is powered by a new cross-platform [Octopus client](https://github.com/OctopusDeploy/go-octopusdeploy) written in [Go](https://golang.org). This client is useful as it allows teams to easily interact with Octopus without direct calls to the Octopus REST API with. It complements the [Octopus CLI](https://octopus.com/docs/octopus-rest-api/octopus-cli) and other Octopus [API clients](https://octopus.com/docs/octopus-rest-api).

### Prerequisites

You need the following to use the Octopus Terraform provider.

* Octopus Deploy 2019.1 or newer
* Go 1.12 or newer
* Terraform 0.12 or newer

### Installing the Octopus Terraform Provider

The Octopus Terraform provider is published in the Terraform Registry so you simply need to delare it in your configuration files to install it.

### Creating your first Terraform script

To get started, I will walk through a simple example that shows you how to add a new variable set to an existing Octopus instance and Space. 

1. **Create Terraform configuration files**

* `main.tf` - This is the main configuration file that configures the provider and specifies resources to create or update. 
* `variables.tf` - This file defines the variables used in the `main.tf` configuration file.
* `terraform.tfvars` - This file contains the values for the variables defined in `variables.tf`.

2. **Configure the main Terraform configuration file.** Open the `main.tf` file and copy/paste the following. The first block configures the Octopus Deploy provider and the second one defines a new variable set resource. 

``` json
provider "octopusdeploy" {
  address  = var.serverURL
  apikey   = var.apiKey
  space_id = var.space
}

resource "octopusdeploy_library_variable_set" "newvariableset" {
  description = var.description
  name        = var.variableSetName
}
```

3. **Add variable definitions.** Open the `variables.tf` file and copy/paste the following variable definitions. The content of this file are straightforward. We are defining variable names and the type of data they will store. In this case, everything is a simple string.

```json
variable "apiKey" {
    type = string
}

variable "space" {
    type = string
}

variable "serverURL" {
    type = string
}

variable "variableSetName" {
    type = string
}

variable "description" {
    type = string
}
```

4. Set your variable values which Terraform passes to your configuration file when the configuration is applied at runtime. Add variable definitions and values. Open the `terraform.tfvars` file and copy/paste the following.  and update the values with your Octopus Server details and a random variable set name and description. 

```json
serverURL       = "https://mytestoctopuscloud.octopus.app"
space           = "Spaces-1"
variableSetName = "AWS configuration values"
description     = "Collection of common AWS configuration values for multiple automated deployment and runbook processes."
```

5. Create our new Octopus Variable set resource by applying the Terraform configuration file to our Octopus instance. This is done by running the following commands at a terminal or command prompt to initialize, verify and apply your changes.

* `terraform init`
* `terraform plan`
* `terraform apply`

NOTE: This is not an exhaustive Terraform tutorial. I highly recommend you read the excellent Terraform documentation to learn more about Terraform itself.

TODO: Screenshot

Congrats! You have using Terraform and the Octopus Deploy provider to configure your Octopus instance. If you navigate to the Library variable sets within your configured space, you will now see your new variable set! 

### Next steps

Checkout the `Examples` folder in the GitHub repo for more examples. 

## Open source and contributing

The [project repository](https://github.com/OctopusDeploy/go-octopusdeploy) is available on GitHub with a Mozilla Public License Version 2.0 license. We accept pull requests and we'd love to see community contributions.

## Configuration as Code vs Infrastructure as Code with the Terraform provider

We're currently building support for [Configuration as Code](https://octopus.com/blog/shaping-config-as-code) to enables teams to have a version-controlled text representation of an Octopus project. This brings numerous benefits but it also raises several questions. What is the difference between Configuration as Code and Infrastructure as Code with our Terraform Provider? When should I use Configuration as Code? When should I use the Terraform provider?

Both technologies are valuable and they are complementary. You can choose to use one or both features depending on your team's needs. The biggest difference between the two technologies is scope. Infrastructure as code focuses on provisioning and configuring a whole Octopus instance whereas Configuration as Code focuses on automated processes in a project.

With **Configuration as Code**, you get a human-readable version of an automated process (deployment and runbook) in Git source-control. This brings numerous benefits including capturing history, enabling changes with branches, having a single source of truth and improving the ability to create template configurations than can be cloned. This is focused on automated processes in a project. It does not allow you to configure other areas of Octopus.

With **Infrastructure as Code**, you can provision new Octopus instances as well as configuring existing ones. This covers most of the Octopus system from infrastructure create and manage. You also gain numerous benefits from the Terraform ecosystem including consistent for configuring infrastructure as well as apply changes and detecting drift.



## Conclusion

We're thrilled to share our Terraform provider and we hope it helps teams manage their Octopus instances with a declarative "Infrastructure as Code" approach. 