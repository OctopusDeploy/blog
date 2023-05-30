---
title: Exploring the managed instance and space patterns
description: Learn how to implement the managed instance and space patterns in Octopus
author: matthew.casperson@octopus.com
visibility: public
published: 2023-01-07-1400
metaImage: blogimage-microservicesframeworks-2022.jpg
bannerImage: blogimage-microservicesframeworks-2022.jpg
bannerImageAlt: People building an unstable tower with blue blocks, beside 2 people building a stable, lower tower with blue blocks.
isFeatured: false
tags:
 - DevOps
 - Enterprise
---

In the [previous post](https://octopus.com/blog/reference-implementation/), you learned how to deploy the enterprise patterns reference implementation. This created and configured an Octopus instance complete with all the resources to show the "managed space per business unit/application" and "managed instance per business unit/region" patterns. 

In this post, we'll walk through the implementation of these patterns.

## Support Levels

The process presented in this post is part of a pilot program to support platform engineering teams. It incorporates tools with varying levels of official support. The Octopus support team will make reasonable endeavors to help customers that wish to use this process. However, existing support Service Level Agreements (SLAs) do not apply to many of the tools described below.

:::warning 
This post describes tools and processes we're seeking feedback on. They're not covered by existing SLAs, and you shouldn't apply them to production systems, or rely on them for production deployments. 
:::

## Managed spaces versus managed instances

The enterprise patterns distinguish between the concept of a managed space and a managed instance. This is because spaces have subtle, yet significant, differences in functionality to instances. Here are the specific differences:

- Spaces on a shared instance all share the same task queue, whereas multiple instances have their own independent task queues.
- An individual Octopus instance (standalone or part of a HA cluster) must have a low latency connection to the database. This effectively limits the physical location of an Octopus instance, and the spaces it contains, to a single location. You need multiple Octopus instances to co-locate regionally disperse users and targets with an Octopus instance.
- It's often easier to satisfy the security requirements of standards like PCI using separate Octopus instances.

However, from a management point of view, it often makes no difference if you manage a separate space or a separate instance. The Octopus REST API performs all management, which means a space or an instance is simply represented by a URL and API key. There are, of course, many subtle differences that may complicate multi-instance management, including things like regional networking and firewalls. But the processes described in this post apply equally well to managing multiple spaces and multiple instances. And the reference implementation demonstrates the management of multiple spaces via configurable URLs and API keys that you can modify to support multiple instances.

## Managed instance per environment vs managed spaces/instances

The "managed instance per environment" pattern describes the process of promoting projects between spaces representing environments like development, test, and production. Teams use this pattern to validate their project changes in a development environment, and they promote the changes to spaces or instances representing more stable environments.

In practice, the "managed instance per environment" pattern is a specialization of the "managed space per business unit/application" and "managed instance per business unit/region" patterns, where you model each environment as a space or instance. The first environment (typically called development) takes the role of the management space, where you iterate on template projects. After you're ready to promote the changes, you apply them to the subsequent environments (such as test and production), which you treat as managed spaces.

The **Development** and **Test/Production** spaces in the reference implementation provide an example of the "managed instance per environment" pattern.

## Library variable sets and tenants representing managed spaces

The first step to managing spaces and instances is to capture the details required to connect to the Octopus REST API. To do this, we'll use a combination of library variable set variable templates and tenants.

Variable templates in library variable sets provide a way to define a set of variables that a tenant must define. We'll create a library variable set called `Octopus Server` with 3 variable templates:

- `ManagedTenant.Octopus.Url` capturing the URL of the Octopus instance hosting the managed space.
- `ManagedTenant.Octopus.ApiKey` capturing the API key used to access the REST API.
- `ManagedTenant.Octopus.SpaceId` capturing the ID of the managed space.

![Library variable set variable templates screenshot](octopus-server-library-variable-set.png "width=500")

Each managed space is then represented by a Tenant. Tenants connected to projects that link the `Octopus Server` library variable set then have the ability to define values for the variable templates.

:::hint
You must first connect a tenant to a project that includes the library variable set before you can define the values for the tenant variables.
:::

The next step is to define tenants for each managed space. The reference implementation defines two regional tenants called **America** and **Europe**:

![Tenants screenshot](tenants.png "width=500")

Note that you may not know the `ManagedTenant.Octopus.SpaceId` value just yet, as the space may not exist. This is fine, and you can define a placeholder value before you know the final value:

![Tenant variables screenshot](tenant-variables.png "width=500")

## Creating managed spaces

The next step is to create a runbook to create managed spaces for each tenant. The reference implementation defines a tenanted runbook called **Create Client Space** in the **__ Create Client Space** project.

:::hint
The double dash prefix on project and runbook names is a convention used by the reference implementation to group these resources in the UI and distinguish them from the template projects.
:::

This runbook makes use of the [Octopus Terraform provider](https://registry.terraform.io/providers/OctopusDeployLabs/octopusdeploy/latest/docs) to create the new space as part of a **Apply a Terraform template** step.

See the Terraform configuration to create a new space below:

:::hint
Note that the Terraform configuration files below omit the required providers and backend settings. The reference implementation uses a Postgres database to hold the Terraform form state, although any [backend](https://developer.hashicorp.com/terraform/language/settings/backends/configuration) is suitable.

See below for an example of the required provider and backend configuration:

```
terraform {
  required_providers {
    octopusdeploy = { source = "OctopusDeployLabs/octopusdeploy", version = "0.12.1" }
  }

  backend "pg" {
    conn_str = "postgres://terraform:terraform@terraformdb:5432/database_name?sslmode=disable"
  }
}
```
:::

 <script src="https://emgithub.com/embed-v2.js?target=https%3A%2F%2Fgithub.com%2FOctopusSolutionsEngineering%2FEnterprisePatternsReferenceImplementation%2Fblob%2Fmain%2Fshared%2Fspaces%2Foctopus%2Fterraform.tf&style=default&type=code&showBorder=on&showLineNumbers=on&showFileMeta=on&showCopy=on"></script>

You define the Terraform variables with the following values:

- `octopus_server` set to `#{ManagedTenant.Octopus.Url}`
- `octopus_apikey` set to `#{ManagedTenant.Octopus.ApiKey}`
- `space_name` set to `#{Octopus.Deployment.Tenant.Name}`

The Terraform workspace is also set to the tenant name of `#{Octopus.Deployment.Tenant.Name}`:

![Screenshot of the create space Terraform step](create-space-step.png "width=500")

After you create the space, the new space ID must be set back in the tenant variables. If you recall from earlier, this was set to a dummy value because you didn't know the value.

The Terraform configuration below sets the value of the `ManagedTenant.Octopus.SpaceId` tenant variable:

 <script src="https://emgithub.com/embed-v2.js?target=https%3A%2F%2Fgithub.com%2FOctopusSolutionsEngineering%2FEnterprisePatternsReferenceImplementation%2Fblob%2Fmain%2Fmanagement_instance%2Fprojects%2Fcreate_client_space%2Fembedded_terraform%2Ftenant_space_id_variable.tf&style=default&type=code&showBorder=on&showLineNumbers=on&showFileMeta=on&showCopy=on"></script>

Terraform output variables are automatically converted to [Octopus output variables](https://octopus.com/docs/projects/variables/output-variables) . This allows subsequent steps to consume the value. This lets you pass the `space_id` Terraform output variable, captured as the Octopus output variable `#{Octopus.Action[Create Client Space].Output.TerraformValueOutputs[space_id]}`, to the `space_id` variable in the configuration above:

![Screenshot showing the Terraform variables being set](set-space-id-step.png "width=500")

## Populating managed spaces

After you create the space, you must populate it with common global resources required to support the template projects that you'll eventually create in the space. At a minimum, these global resources include:

- Environments
- Lifecycles
- Git Credentials
- Feeds

This Terraform configuration creates the standard environments:


 <script src="https://emgithub.com/embed-v2.js?target=https%3A%2F%2Fgithub.com%2FOctopusSolutionsEngineering%2FEnterprisePatternsReferenceImplementation%2Fblob%2Fmain%2Fshared%2Fenvironments%2Fdev_test_prod%2Foctopus%2Fterraform.tf&style=default&type=code&showBorder=on&showLineNumbers=on&showFileMeta=on&showCopy=on"></script>

This Terraform configuration creates the lifecycle:

 <script src="https://emgithub.com/embed-v2.js?target=https%3A%2F%2Fgithub.com%2FOctopusSolutionsEngineering%2FEnterprisePatternsReferenceImplementation%2Fblob%2Fmain%2Fshared%2Flifecycles%2Fsimple_dev_test_prod%2Foctopus%2Fterraform.tf&style=default&type=code&showBorder=on&showLineNumbers=on&showFileMeta=on&showFullPath=on&showCopy=on"></script>

This Terraform configuration creates the shared Git credentials:

 <script src="https://emgithub.com/embed-v2.js?target=https%3A%2F%2Fgithub.com%2FOctopusSolutionsEngineering%2FEnterprisePatternsReferenceImplementation%2Fblob%2Fmain%2Fshared%2Fgitcreds%2Fgitea%2Foctopus%2Fterraform.tf&style=default&type=code&showBorder=on&showLineNumbers=on&showFileMeta=on&showFullPath=on&showCopy=on"></script>

This Terraform configuration creates the DockerHub feed:

 <script src="https://emgithub.com/embed-v2.js?target=https%3A%2F%2Fgithub.com%2FOctopusSolutionsEngineering%2FEnterprisePatternsReferenceImplementation%2Fblob%2Fmain%2Fshared%2Ffeeds%2Fdockerhub%2Foctopus%2Fterraform.tf&style=default&type=code&showBorder=on&showLineNumbers=on&showFileMeta=on&showFullPath=on&showCopy=on"></script>

These resources are the minimum required to support the projects that you'll deploy to the managed space to support a simple "Hello World" style deployment project.

You will note, however, that the reference implementation goes beyond these steps to configure resources like teams, additional feeds, project groups, and library variable sets. These additional resources support more complex deployments to cloud platforms with associated runbooks to automate common management and support tasks.

Executing the **Create Client Space** runbook against a tenant creates the new space on the managed Octopus instance, complete with all the common global resources required to support future managed projects. Here we see the **America** managed space with the global environments created:

![A screenshot of the environments created in a managed space](managed-space.png "width=500")

## Creating a template project

You're now ready to create the template of a project to deploy to your managed spaces. The sample project, called `Hello World`, defines 2 script steps and defines a handful of variables to demonstrate common scenarios that teams will likely encounter when deploying projects to managed spaces.

### Deciding whether to template the templates

The reference implementation creates the template projects in the **Default** space with Terraform, and you can see the configuration files required to create the `Hello World` upstream project below. The reference implementation had no choice but to "template the templates" as we designed it to create a complete stack from scratch.

However, it's up to you if you want to create these upstream projects by hand or define them with Terraform.

The purpose of the upstream projects is two-fold:

1. To provide a source Octopus resource to be serialized into a Terraform module.
2. To provide the ability to edit and test projects directly in the Octopus UI before deploying them to managed spaces.

There's no requirement that you manage these template projects with Terraform. I'd expect most teams to manually create and edit these projects in Octopus.

### Creating the Hello World project

The project will reference a library variable set called `Export Options`. The purpose of this library variable set is to define variables used by the management runbooks (which we'll cover later). But you shouldn't include that when serializing the project to a Terraform module.

You use a library variable set rather than a project variable because it's trivial to exclude a library variable set when exporting a project. However, for Config as Code enabled projects, it's far more difficult to exclude individual project variables as they're committed directly to a Git repository and inherited by any forked repositories.

This is the Terraform configuration to create the `Export Options` library variable set:

<script src="https://emgithub.com/embed-v2.js?target=https%3A%2F%2Fgithub.com%2FOctopusSolutionsEngineering%2FEnterprisePatternsReferenceImplementation%2Fblob%2Fmain%2Fshared%2Fvariables%2Fexport_options%2Foctopus%2Fterraform.tf&style=default&type=code&showLineNumbers=on&showFullPath=on&showCopy=on&fetchFromJsDelivr=on"></script>

![Export Options variable set screenshot](export-options-variable-set.png "width=500")

This is the Terraform configuration to create the sample project:

:::hint
The secret variables defined in the project follow some rules that allow them to be replicated when deployed to managed spaces. We explain the reasoning behind this in the "Creating the management runbooks" section.
:::

<script src="https://emgithub.com/embed-v2.js?target=https%3A%2F%2Fgithub.com%2FOctopusSolutionsEngineering%2FEnterprisePatternsReferenceImplementation%2Fblob%2Fmain%2Fmanagement_instance%2Fprojects%2Fhello_world%2Foctopus%2Fterraform.tf&style=default&type=code&showLineNumbers=on&showFullPath=on&showCopy=on&fetchFromJsDelivr=on"></script>

The end result of applying these Terraform configuration files is a simple project called `Hello World` with 2 steps:

![Screenshot of hello world project](hello-world.png "width=500")

It's important to understand that this template project isn't special. You can execute and test it in the management space and edit the steps and variables like normal.

The ability to serialize this project to a Terraform module and deploy it to managed spaces is implemented by the management runbooks attached in the next section.

## Creating the management runbooks

You require 2 runbooks to turn this regular project into a template project:

1. The first serializes the project to a Terraform module.
2. The second deploys the Terraform module to a managed space.

The first runbook, called `__ 1. Serialize Project`, does the following:

1. Executes an open source tool called [octoterra](https://github.com/OctopusSolutionsEngineering/OctopusTerraformExport) to serialize the project to a Terraform module.
2. Packages the Terraform module as a zip file with the Octopus CLI.
3. Pushes the zip file to the management space built-in feed.

The Python script shown below implements this process:

<script src="https://emgithub.com/embed-v2.js?target=https%3A%2F%2Fgithub.com%2FOctopusSolutionsEngineering%2FEnterprisePatternsReferenceImplementation%2Fblob%2Fmain%2Fmanagement_instance%2Frunbooks%2Fshared_scripts%2Fserialize_project.py&style=default&type=code&showLineNumbers=on&showFullPath=on&showCopy=on&fetchFromJsDelivr=on"></script>

The Terraform module created by `octoterra` exposes 3 variables you need to define when applying the module:

1. `octopus_server` which defines the Octopus server that Terraform will connect to.
2. `octopus_space_id` which defines the space ID that Terraform will populate.
3. `octopus_apikey` which defines the Octopus API key Terraform uses when interacting with the Octopus API.

`octoterra` implements a number of conventions to allow the downstream project to be customized as it's applied.

An example of this is an optional variable called `project_hello_world` which can be overridden to define the name of the downstream project. If this variable is not defined, it defaults to the name of the upstream project.

:::hint
The name of the variable defining the name of the project has the prefix of `project_` and the suffix of the upstream project name converted to lowercase and with every non-alphanumeric character replaced with an underscore.
:::

Another example is that sensitive values are defined in files starting with the prefix `project_variable_sensitive`. This is significant because Octopus can replace Octostache template syntax in selected files before applying the Terraform module. You take advantage of this by passing `octoterra` the `-defaultSecretVariableValues` argument, which sets the default value of any secret variable to the Octostache syntax referencing the project variable. 

For example, this is the value of the `project_variable_sensitive_hello_world_database_development__password_1.tf` file:

```hcl
variable "hello_world_database_development__password_1" {
  type        = string
  nullable    = false
  sensitive   = true
  description = "The secret variable value associated with the variable Database[Development].Password"
  default     = "#{Database[Development].Password}"
}

resource "octopusdeploy_variable" "hello_world_database_development__password_1" {
  owner_id        = "${octopusdeploy_project.project_hello_world.id}"
  name            = "Database[Development].Password"
  type            = "Sensitive"
  description     = ""
  sensitive_value = "${var.hello_world_database_development__password_1}"
  is_sensitive    = true

  scope {
    actions      = []
    channels     = []
    environments = ["${data.octopusdeploy_environments.environment_development.environments[0].id}"]
    machines     = []
    roles        = null
    tenant_tags  = null
  }
  depends_on = []
  lifecycle {
    ignore_changes = all
  }
}
```

Note how the Terraform variable defines the value of the Octopus variable. This is another convention that lets you customize the template project when it's applied. Also, note the default value for the Terraform variable is set to the Octostache template syntax `#{Database[Development].Password}`.

This convention works around the fact that the Octopus API never exposes the value of secret variables. Because all the tooling used to implement the enterprise patterns accesses Octopus via its API, the value of secret variables are not directly accessible. However, by embedding Octostache template syntax into the exported Terraform modules, you can have Octopus inject secret values when applied with the **Apply a Terraform template** step.

This is why secret variables must be unambiguous (that is, only have a single value) and you must scope them to the **Sync** environment. Doing so allows the `Apply a Terraform template` step run in the **Sync** environment to correctly link up secret values in the exported project Terraform modules.

You will also note that `octoterra` was called with the `-excludeVariableEnvironmentScopes` argument set to `Sync`. This argument excludes any variable environment scope set to the **Sync** environment. This results in the exported Octopus variable having an environment scope with all environments except **Sync**, meaning the downstream space does not need to have a **Sync** environment.

:::hint
`octoterra` has many arguments to exclude certain Octopus resources from the generated Terraform module. It also exposes many Terraform variables allowing aspects of the exported project to be customized when it's applied.
:::

The second runbook, called **__ 2. Deploy Project**, is a tenanted runbook that applies the Terraform module created by the first runbook via a **Apply a Terraform template** step with the following settings:

1. The `Replace variables in default Terraform files` option is disabled.
2. Variables are replaced in files matching the name `**/project_variable_sensitive*.tf`.
3. The Terraform workspace `#{Octopus.Deployment.Tenant.Name | ToLower | Replace "[^a-zA-Z0-9]" "_"}_#{Exported.Project.Name | ToLower | Replace "[^a-zA-Z0-9]" "_"}` is used.
4. The apply parameters are set to `-var="octopus_server=#{ManagedTenant.Octopus.Url}" -var="octopus_space_id=#{ManagedTenant.Octopus.SpaceId}" -var="octopus_apikey=#{ManagedTenant.Octopus.ApiKey}" -var="project_#{Octopus.Project.Name | ToLower | Replace "[^a-zA-Z0-9]" "_"}_name=#{Exported.Project.Name}"`.

Note how the variables associated with a tenant (`ManagedTenant.Octopus.Url`, `ManagedTenant.Octopus.SpaceId`, and `ManagedTenant.Octopus.ApiKey`) allow you to deploy the project to any Octopus space, while the variables defined in the `Export Options` library variable set (`Exported.Project.Name`) allow you to customize the name of the downstream project.

:::hint
The reference implementation executes a number of additional steps when deploying the **__ 2. Deploy Project** runbook to:

1. Create a Postgres database to hold the Terraform state.
2. Trigger the creation of the manage space runbook to ensure it exists.
3. Compose any specialized space resources (this is unused in the case of the `Hello World` project)
4. Query the space ID associated with the tenant, as new variables assigned to a tenant in step 2 are not injected into an already running process.
5. Deploy the Terraform module.
:::

These 2 runbooks are all you need to turn a regular non-Config as Code project into a template project. 

Running the **__ 1. Serialize Project** runbook creates a Terraform module and uploads it to the built-in feed:

![Screenshot of the terraform module uploaded to the built-in feed](terraform-module-package.png "width=500")

Running the **__ 2. Deploy Project** runbook applies the Terraform module to the managed space represented by the tenant:

![Screenshot of the downstream project in the America space](managed-project.png "width=500")

## Creating Config as Code Hello World project

The `Hello World` project created in the previous section is an example of a regular non-Config as Code enabled Octopus project. Config as Code enabled projects let you take advantage of Git-based workflows for template projects, but require some additional steps to your management runbooks to populate the downstream templates.

To demonstrate a Config as Code (CaC) template project, create a project called `Hello World CaC` with 2 steps:

1. A manual intervention step that displays a prompt when a deployment is initiated in the **Production** environment.
2. A simple script step printing some text to the logs.

![A screenshot of the CaC enabled Hello World project](hello-world-cac.png "width=500")

Again, there's nothing special about this project. It is a regular Config as Code enabled project, and the deployment process, variables, and other settings can be freely edited in the Octopus UI.

As before, we have 2 management runbooks to serialize the project to a Terraform module and apply the module to the managed space.

The first runbook, **__ 1. Serialize Project**, is the same as described in the previous section.

The second runbook, called **__ 2. Fork and Deploy Project**, does the following:

1. Fork the upstream Config as Code Git repository into a new downstream Git repository.
2. Apply the Terraform module created by the **__ 1. Serialize Project** runbook.

The process of forking a Git repository depends on the git platform you're using. It usually requires creating a new Git repository, adding a new remote upstream to reference the upstream repository, performing a hard reset on the downstream repository, and pushing the results.

The script below runs through the process for repositories hosted by Gitea:

<script src="https://emgithub.com/embed-v2.js?target=https%3A%2F%2Fgithub.com%2FOctopusSolutionsEngineering%2FEnterprisePatternsReferenceImplementation%2Fblob%2Fmain%2Fmanagement_instance%2Frunbooks%2Fshared_scripts%2Ffork_repo.py&style=default&type=code&showBorder=on&showLineNumbers=on&showFileMeta=on&showFullPath=on&showCopy=on"></script>

:::hint
The script shown above is only suitable for Gitea. Other platforms, like GitHub, GitLab, BitBucket, etc, need their own custom implementation of this script to fork repositories.
:::

After you fork the repository, you can apply the template project can in the managed space. This is largely similar to the process described in the previous section, using the **Apply a Terraform template** step to apply the Terraform module created by the **__ 1. Serialize Project** runbook. One difference is the URL of the Config as Code repository is overridden to point to the newly forked repository by passing the argument `-var="project_#{Octopus.Project.Name | ToLower | Replace "[^a-zA-Z0-9]" "_"}_git_url=http://gitea:3000/octopuscac/#{Octopus.Deployment.Tenant.Name | ToLower}_#{Exported.Project.Name | ToLower | Replace "[^a-zA-Z0-9]" "_"}.git"`.

:::hint
The URL of the forked repository passed in the argument above hardcodes the hostname of the local Gitea platform. You must modify this when using other Git platforms.
:::

:::hint
The **__ 2. Fork and Deploy Project** runbook in the reference implementation adds the same additional steps noted for the **__ 2. Deploy Project** runbook in the previous section.
:::

As before, these 2 runbooks are all you need to turn a regular non-Config as Code project into a template project. 

Running the **__ 1. Serialize Project** runbook creates a Terraform module and uploads it to the built in feed:

![Screenshot of the terraform module uploaded to the built-in feed](terraform-module-cac-package.png "width=500")

Running the **__ 2. Fork and Deploy Project** runbook forks the Git repository and applies the Terraform module to the managed space represented by the tenant:

![Screenshot of the downstream project in the America space](managed-cac-project.png "width=500")

## Conclusion

In this post, we look at how to represent managed spaces with library variable sets and tenants. We created managed spaces based on the tenants, complete with shared global resources like environments, feeds, git credentials, and project groups. We then created a regular and Config as Code upstream project and attached runbooks that serialized them to Terraform modules, forked the Git repository for Config as Code enabled projects, and applied the Terraform modules to the managed spaces.

This workflow supports very simple implementations of the enterprise patterns where you create template projects push them to managed spaces. However, a more complete implementation of the enterprise patterns requires addressing the full lifecycle of managed spaces, including pushing additional changes made to upstream projects and allowing you to edit downstream projects.

In the [next post](https://octopus.com/blog/synchronizing-projects), we'll dive deeper into how you can maintain projects and synchronize them over time.

We're currently refining our approach to these enterprise patterns, so if you have any suggestions or feedback about the approach described here, please leave a comment on [this GitHub issue](https://github.com/OctopusSolutionsEngineering/EnterprisePatternsReferenceImplementation/issues/1).

Happy deployments!