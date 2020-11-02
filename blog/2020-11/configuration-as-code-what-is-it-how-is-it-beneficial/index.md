---
title: "Configuration as Code: What is it and how is it beneficial?"
description: Learn about the benefits of Configuration as Code and some of the considerations to understand when implementing Configuration as Code.
author: adbertram@gmail.com
visibility: public
published: 2020-11-02
metaImage: blogimage-config-as-code-explanation_2020
bannerImage: blogimage-config-as-code-explanation_2020
tags:
 - DevOps
 - Config as Code
---

![Configuration as Code: What is it and how is it beneficial?](blogimage-config-as-code-explanation_2020)

Managing application configuration settings is becoming an increasingly important aspect of modern application development. Typically, configurations are stored with their associated application code repositories and any changes necessitate a new version of the code to be deployed. This is true, even if only a single configuration setting is changed!

Configuration as Code (CaC) treats your application settings as first-class citizens. Often, this implies that the configurations are stored in their own repository and managed in a different process from the primary codebase.

In this article, we explore what exactly Configuration as Code means and what benefits are derived by promoting configurations to first-class citizens in the DevOps process.

## Benefits of Configuration as Code

Configuration as Code, as generally outlined above, is the versioned migration of configurations between different environments. Other than adding complexity to existing build and deployment processes, what does this approach gain an organization?

- **Security**: User access separation promotes least-privilege permissions and higher levels of access auditability.
- **Traceability**: Using a separate version control repository, specific to configurations, makes it easier to trace changes and updates.
- **Manageability**: Build and deployment processes specific to configurations can include additional approval, validation, and testing steps to ensure non-breaking changes.

As you can see, there is a myriad of benefits associated with moving configuration management into its own unique process. Though complexity is introduced, and some setup and configuration is needed to remove configuration from within a codebase, the benefits in the long run, are well worth the effort.

Taking the time to plan a Configuration as Code approach that ensures the configuration is versioned, accessible to only those who need it, secrets are encrypted, and proper approval gates are in place will ensure a successful and managed configuration deployment process.

## Infrastructure as Code vs. Configuration as Code

Infrastructure as Code (IaC) concerns itself with the deployment of the underlying infrastructure necessary to support an application’s environment. This can lead to confusion as it looks similar to managing configurations. Configuration as Code is about managing the specific application configuration settings themselves, which is separated from your infrastructure code and managed in its own unique process.

That is not to say that both Infrastructure as Code and Configuration as Code do not complement each other and are often used hand in hand. For example, certain configurations can, and should, be managed in the configuration process that will later be consumed by the infrastructure processes. By using each approach in this way, an organization can quickly get a handle on complex configurations and how they are managed.

## Practical use of configuration as code

What does using Configuration as Code in practice actually imply? There are a number of ways to implement this approach in practice, and not all of them make sense for every organization. Read how the broad strokes outlined below may fit your unique needs.

- Using unique configuration source control repositories.
- Developing a tailored build and deployment process.
- Creating configuration specific test environments.
- Ensuring approval and quality assurance processes exist.
- Secrets management within configurations.

### Using unique configuration source control repositories

Typically, application configuration is packaged with the same code that is deployed. When a configuration value is changed, the entire codebase must then be deployed to effect the change. Although this means there is a versioned change, the configuration settings are normally stored in a flat file, and it can be difficult to trace the exact change made, especially if done in a larger code commit.

By storing and versioning configuration in its own repository, you gain the benefit of independent configuration changes, a tailored deployment process, and ease of change audits. Furthermore, additional security controls can be put into place, which are usually used for application secrets that a developer may not need to have access to.

### Developing a tailored build and deployment process

The deployment process for an application may not fit the needs of simple configuration changes. Often the configuration build and deployment process is simplified. The validation and testing of a configuration change may not touch as many different aspects of the application that a code change may.

With a specific configuration deployment process, additional validation steps can be built into the process. This can eliminate many of the process steps that may not make sense for configuration specific deployments. For example, configuration items may need to be validated, such as format and substance, which can then be handled in a configuration specific build and deployment process.

### Creating configuration specific test environments

Perhaps spinning up a full application code testing environment is not needed for a simple configuration change. The ability to scope a test environment just to the needs of the configuration deployment process can save time and money for an organization.

This may also mean that independent changes can happen at the same time. Application developers can test their code while a configuration change is tested as well. With this ability for parallel testing, you gain efficiency in the operation and management of the environment.

### Ensuring approval and quality assurance processes exist

Perhaps changing an API key or endpoint URI will need a different team’s input and sign-off. This approval process would not typically exist if the configurations themselves were buried within the application codebase. By dovetailing the tailored build and deployment process, both approvals and quality assurance can be integrated to ensure that just the right configurations are deployed where they should be.

### Secrets management within configurations

Most configurations will include secrets necessary to access databases, key stores, or file locations that should not be widely shared. With configurations that live solely in a flat file stored next to application code, it can be difficult to secure those secrets to only those who should have access. With Configuration as Code, you can separate these protected secrets to encrypted values that are subject to different approval and verification processes. This increases the security of your applications and greatly decreases the chance of code or infrastructure breaches.

## Conclusion

Though there are many different ways to integrate DevOps and all of its associated processes into your environment, Configuration as Code makes a great starting point to work towards a more managed and controlled application environment.

The gain in security, auditibility, manageability, and control afforded to organizations integrating Configuration as Code into their workflows make managing complex configurations easier and more secure than with the typical approach to bundling configuration within an application’s codebase.
