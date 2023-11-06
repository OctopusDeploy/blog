---
title: Terraform Spaces per Resources
description: Spaces Support on Resources for Octopus Deploy Terraform Provider.
author: domenic.simone@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage:
bannerImage:
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags:
  - DevOps
  - Terraform
  - Configuration as Code
---

## Body

In the past, the Octopus Deploy Terraform Provider had Spaces scoped per provider configuration; this can cause all sorts of hurdles when creating resources across multiple spaces. Today, we are thrilled to announce that the Octopus Terraform Provider now supports Spaces scoped per resource.

### The Issue with Spaces per Provider Configuration

Defining a Space on the provider level is viable, especially when creating many resources within the same Space. However, the issue arises when trying to create resources across multiple Spaces, as a new provider with an alias will need to be created for each Space, as shown in the example below. This approach can be very counterproductive and potentially have significant limitations.

```terraform
provider "octopusdeploy" {
  alias = "space_one"
  address = "https://octopus.example.com"
  api_key = "API-XXXXXXXXXXXXX"
  space_id = "Spaces-1"
}

provider "octopusdeploy" {
  alias = "space_two"
  address = "https://octopus.example.com"
  api_key = "API-XXXXXXXXXXXXX"
  space_id = "Spaces-2"
}

resource "octopusdeploy_username_password_account" "account1" {
  provider = octopusdeploy.space_one
  name = "Test Account"
  password = "XXXX"
  username = "testusernameone"
}

resource "octopusdeploy_username_password_account" "account2" {
  provider = octopusdeploy.space_two
  name = "Test Account"
  password = "XXXX"
  username = "testusernametwo"
}
```

### The Benefit of Spaces per Resource

To significantly improve the experience, we have now introduced the concept of a Space per resource. This approach allows each resource to explicitly specify the Space in which it should be created. If no Space is set on the resource, we will seamlessly revert back to using the Space defined at the provider level. Below is the same example as mentioned earlier, but now we are leveraging the concept of Space per resource for enhanced flexibility and control.

```terraform
provider "octopusdeploy" {
  address = "https://octopus.example.com"
  api_key = "API-XXXXXXXXXXXXX"
  space_id = "Spaces-1"
}

resource "octopusdeploy_username_password_account" "account1" {
  name = "Test Account"
  password = "XXXX"
  username = "testusernameone"
}

resource "octopusdeploy_username_password_account" "account2" {
  space_id = "Spaces-2"
  name = "Test Account"
  password = "XXXX"
  username = "testusernametwo"
}

```

### When will Spaces per Resource be available?

It is available now in Octopus Deploy Terraform Provider version 0.13 or newer.

## Conclusion

The evolution of the Octopus Deploy Terraform Provider from Spaces scoped per provider configuration to Spaces scoped per resource marks a significant enhancement in the usability and flexibility of the terraform platform. Before, managing resources in different Spaces was tricky and required creating separate provider configurations for each Space. Now, it's much simpler. You can assign a Space to each resource individually without complex configurations. This change gives you more control and makes the whole process easier.

