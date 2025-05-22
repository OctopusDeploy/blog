---
title: Generating projects with GenAI
description: Learn about how we're using GenAI to generate projects in Octopus Deploy.
author: matthew.casperson@octopus.com
visibility: provate
published: 2025-06-01-1400
metaImage: img-blog-laptop-cogs-cloud.png
bannerImage: img-blog-laptop-cogs-cloud.png
bannerImageAlt: Stylized laptop screen showing Octopus logo connected to cogs in the cloud, with a clipboard to the right.
tags:
  - Product
isFeatured: false
---

## Introduction

The DevOps landscape is growing increasingly complex. Cloud providers regularly add new platforms, the Kubernetes ecosystem is undergoing a Cambrian explosion, and all of this while DevOps teams are expected to address security, observability, scale, and compliance in every step of their processes.

Octopus has an amazing ability to model the complex environments found in modern enterprises, but configuring the environments, feeds, accounts, lifecycles, projects, runbooks, and targets that make up any real world deployment scenario can be a challenge, let alone doing so according to best practises with patterns that will scale.

To help teams get up and running quickly on Octopus, we're introducing a new feature that populates an Octopus space with common projects based on a prompt like `Create a Kubernetes project called "Web App"`. This functionality is powered by GenAI, and in this post I'll describe how a simple prompt becomes a fully functional project in your Octopus instance.

## Building the templates

GenAI is only as good as the data it has learned from. For GenAI to build a project, along with all the supporting resources, we start by hand crafting a number of template projects according to a set of best practices that we've evolved over the years. 

At this stage the template projects are nothing more than regular projects manually created via the web UI. This makes it easy to build and test the project as if it were any other project in Octopus.

Best practices capture aspects like variable and target tag naming conventions, release versioning schemes, worker images, supporting runbooks, and much more. There are hundreds of configurable fields in an Octopus project, and we have a continually evolving set of opinions to provide DevOps teams with a solid foundation for their deployment workflows.

Importantly, we do not train an LLM on any customer data.

Once we have a template project that embodies our best practices, the next step is to serialize the project to a format that can be passed to an LLM.

## Serializing the template

LLMs excel at generating text, and so to have the LLM generate an Octopus project, we need to define projects in a text-based format. We use Terraform, and specifically Terraform implementing the Octopus Terraform provider, to represent the project.

Terraform is a natural choice because:

* We can reasonably expect any large LLM to have been trained on a large body of Terraform examples. This is easy to verify by asking the LLM to generate Terraform code for a popular platform like AWS or Azure.
* HCL files contain multiple resources and express their relationships via interpolations.
* It is trivial to determine result is semantically valid Terraform configuration.

[Octoterra](https://github.com/OctopusSolutionsEngineering/OctopusTerraformExport) is used to convert an existing Octopus project into Terraform configuration. This process is automated, providing a short feedback loop where projects can be updated, serialized, and supplied as an example as part of a LLM prompt.

That said, we could not generate a typical Terraform representation of a project. Terraform relies on persistent external state to track the resources it manages, and we wanted to avoid persisting any information about an Octopus instance were possible. Treating the Octopus instance as the single source of truth meant all data stayed securely behind the Octopus API, removed issues around keeping external data in sync, and removed the security concerns around storing and accessing potentially sensitive data.

To support these requirements, we generated what we call "Stateless Terraform" modules. These modules always pair a data source and the resource to be created in such a way as to create the resource if it didn't exist or reference the existing resource if it was already present. This is an example:

```hcl
data "octopusdeploy_environments" "environment_development" {
  ids          = null
  partial_name = "Development"
  skip         = 0
  take         = 1
}
resource "octopusdeploy_environment" "environment_development" {
  count                        = "${length(data.octopusdeploy_environments.environment_development.environments) != 0 ? 0 : 1}"
  name                         = "Development"
  description                  = ""
  allow_dynamic_infrastructure = true
  use_guided_failure           = false
}
```

This data source and resource pair result in the `count` attribute being set to 1 if the `Development` environment doesn't exist, thus creating the environment, or set the `count` to 0 if the environment is present.

Other resources then use a ternary expression to either reference the existing resource or the newly created one:

```hcl

resource "octopusdeploy_lifecycle" "lifecycle_application" {
  count       = "${length(data.octopusdeploy_lifecycles.lifecycle_devsecops.lifecycles) != 0 ? 0 : 1}"
  name        = "Application"

  phase {
    automatic_deployment_targets = [
      "${length(data.octopusdeploy_environments.environment_development.environments) != 0 ? data.octopusdeploy_environments.environment_development.environments[0].id : octopusdeploy_environment.environment_development[0].id}"
    ]
    optional_deployment_targets = []
    name                                  = "Development"
    is_optional_phase                     = false
    minimum_environments_before_promotion = 0
  }
}
```

This Terraform configuration would not work as expected if it was applied multiple times with a persistent state. Resources would be alternatively created and destroyed as the count associated with a resource toggled between 0 and 1.

However, when the Terraform configuration is run against an empty state, this style of configuration works perfectly. Missing resources are created, and existing resources are left unmodified. This is exactly the behavior we want when generating Octopus resources via the LLM. It provides an imperative method for creating resources if they do not exist without needing to know anything about the Octopus space where the configuration will be applied.

## Training the LLM

Once the ability to serialize projects to stateless Terraform modules is in place, we can pass these modules as part of the context of an LLM prompt. This is called one-shot or few-shot prompting, where the LLM is given an example of the task it is being asked to perform.

These examples got us almost all the way to having an LLM generate a project, and all the supporting resources, from a plain text prompt. However, because the LLMs had not been explicitly trained or fine-tuned on the stateless Terraform modules, we needed some additional prompt instructions to avoid edge cases and steer the LLM towards generating the correct Terraform configuration.

Generating the instructions involved:

* Having an LLM generate multiple sample prompts that we might expect end users to write themselves.
* Running the generated prompts and observing the resulting Terraform configuration.
* Finding cases where the LLM generated invalid output and refining the prompt instructions to avoid these cases.

This is the prompt we used to generate sample prompts:

```text
Given these prompts:

'Create a Kubernetes project called "My K8s Project 2". Add a step to deploy a K8s deployment. Add a step to deploy a Kubernetes ClusterIP service. Use the "Security" lifecycle. Add a step to deploy a Kubernetes ingress resource. Enable variable debugging.'

'Create an Azure Web App project call "My Azure Project 1"`

Write 5 more prompts including a mix of:

feeds
accounts
steps
platforms like helm, kustomize, aws lambdas, azure web apps, azure functions, cloud formation, arm templates, bicep templates
lifecycles
project groups
project names
step run conditions
rollout strategies like blue/green and canary
variables
environments
target tags
packages
cloud providers
retention policies
inline scripts
script from packages
tenanted, untenanted, or tenanted and untenanted deployments
tenants
variable scopes
tenant tags

Do not reuse project names from previous responses.

Include a random number suffix on the project name from 1 to 10000.

Print each example in an individual markdown code block.
```

The generated prompts often requested deployment processes that even an Octopus expert couldn't implement. The goal was not to generate logically valid output, but to ensure our combination of sample Terraform and custom instructions generated semantically valid output. This demonstrated that the LLM had the examples and instructions required to generate valid output, meaning it was up to us to provide the correct sample projects to generate useful output.

You can view the instructions from the [GitHub repository](https://github.com/OctopusSolutionsEngineering/OctopusCopilot/blob/main/context/generalinstructions.txt).

## Generating the project

Once the LLM was trained, the next step was to execute the generated Terraform configuration to populate the Octopus space.

This was a relatively easy task since we did not have to persist any state. So we created an Azure function that:

* Embedded the OpenTofu executable
* Exposed an HTTP endpoint to generate a plan
* Applied the plan once it was approved by the end user

While this is a simple process at a high level, there are a number of security concerns we needed to address:

* The Terraform configuration should only create Octopus resources, not other resources for platforms like Azure or AWS
* We needed to fail if any sensitive data was included in the generated Terraform configuration because the prompt interface is not secure
* We needed to enforce the use of local Terraform state and not allow Terraform to try and save state in any external location
* Any existing resource must not be modified when creating AI generated resources

The first two concerns were addressed by testing the JSON representation of the Terraform plan file with Open Policy Agent. Thanks to the declarative nature of Terraform, it is possible to [determine the provider that will be used to create a resource](https://developer.hashicorp.com/terraform/language/resources/syntax#providers). We can also identify which attributes the provider marks as sensitive and fail unless the sensitive values are set to dummy values like `CHANGE ME`.

To enforce the use of local state we implemented [Terraform override files](https://developer.hashicorp.com/terraform/language/files/override). These files are created alongside the generated Terraform configuration and take precidence over the generated configuration.

Finally, we could take advantage of the fact that Terraform will not modify resources it does not own, and it will never own anything because Terraform was always executed with a blank state.

The combination of OPA policies run against plan JSON files and override files allowed us to restrict the creation of AI-generated resources in a way that would have been all but impossible with custom scripts or raw API calls.

## Customizing the generated project

All of this is overkill if the purpose is to recreate a template project verbatim. Stateless Terraform modules could be applied unmodified with no need to involve an LLM. 

The real power of LLMs are their ability to generate custom responses to any prompt.

The combination of custom LLM instructions, general purpose example Terraform configurations, and specific template project examples allows us to prompt the LLM to generate customized projects. For example, we can write a prompt like this:

```text
Create an Azure Web App project called "My Azure App". 
Create tenant tags based on geographical regions. 
Add 10 tenants named after major capital cities.
Add the tenant tags to the tenants. 
Link the tenants to the project.
```

Because the LLM has seen what a tenant looks like, what tenant tags look like, and how to link tenants to projects, the result is a project based on the template project but with multiple tenants connected to it.

Importantly, this specific scenario is not something we need to train the LLM to do. These customizations to the template projects are possible because of the one-shot and few-shot examples we provided to the LLM.

This allows us to generate complex, bespoke projects that are still based on our best practices. As long as we provide enough examples of valid Terraform configuration, catch edge cases with the LLM instructions, and provide a good set of template projects, end users can generate almost any project they can imagine.

## Creating a Virtuous Cycle

We now had a process that allowed us to generate Octopus projects from hand-crafted template projects. Our engineers could contribute new examples simply by creating an example of a best practice project in Octopus. The process of serializing it to Terraform and associating it was an LLM prompt was either scripted or involved a fairly trivial code change. This created a tight feedback loop that we'll take advantage of to continually improve the quality of the generated projects.

## Conclusion

One of the challenges when integrating AI into an existing platform is identifying where AI adds unique value to solve real-world problems. By training an LLM to generate Octopus projects using their inherent affinity for generating text, we can empower new Octopus users to populate an entire Octopus space with a functional sample project, built on top of our hand-crafted examples, in a matter of minutes. 

This is just one way that we're using GenAI to improve the experience of using Octopus Deploy, and we're excited to see how our customers will use this functionality to accelerate their DevOps processes.