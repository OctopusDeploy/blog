---
title: Terraform spaces per resource
description: Learn about our new spaces support on resources for the Octopus Terraform provider.
author: domenic.simone@octopus.com
visibility: private
published: 2023-11-20-1400
metaImage:
bannerImage:
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags:
  - DevOps
  - Terraform
  - Configuration as Code
---

Until now, the Octopus Deploy Terraform provider had a single space per provider configuration. This caused hurdles when creating resources across multiple spaces. 

We're pleased to announce we've addressed this issue. The Octopus Terraform provider now supports Space IDs per Terraform resource.

In this post, I explain why we made this change and how it makes your process easier.

### Issues with spaces per provider configuration

Defining a space on the provider level does work, especially if you're creating many resources in the same space. 

However, you can face problems when trying to create resources across multiple spaces. This is because a new Terraform provider with an alias gets created for each space, as shown in the example below. This can be counterproductive and limiting. 

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

### Benefits of spaces per resource

To improve your experience, we introduced the concept of a Space ID per Terraform resource. This lets each resource specify the space where it should be created. If no space is set on the resource, we'll seamlessly revert to using the space defined at the provider level. 

Below you can see the same example from above, but now we're using the concept of Space ID per Terraform resource for greater flexibility and control.

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

### When will spaces per resource be available?

The Space ID per Terraform resource feature is available now in Octopus Deploy Terraform provider version 0.13 or newer.

## Conclusion

Updating the Octopus Deploy Terraform provider from a spaces per provider configuration to a Space ID per Terraform resource has improved usability and flexibility.

Before, managing resources in different spaces was tricky. You needed to create separate provider configurations for each space. Now, it's much simpler. You can assign a space to each resource individually without complex configurations. This change gives you more control and makes the process easier.

Happy deployments!