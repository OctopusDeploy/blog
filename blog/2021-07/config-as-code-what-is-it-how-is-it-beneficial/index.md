---
title: "Config as Code: What is it and how is it beneficial?"
description: Learn about the benefits of Config as Code and some of the considerations when implementing Config as Code.
author: adbertram@gmail.com
visibility: public
published: 2021-07-07-1400
metaImage: blogimage-config-as-code-explanation_2020.png
bannerImage: blogimage-config-as-code-explanation_2020.png
bannerImageAlt: Config as Code What is it and how is it beneficial?
tags:
 - DevOps
 - Configuration as Code
---

![Config as Code: What is it and how is it beneficial?](blogimage-config-as-code-explanation_2020.png)

Managing application configuration settings is an increasingly important aspect of modern application development. Typically, configurations are stored with their associated application code repositories and any changes need a new version of the code to be deployed. This is true, even if only a single configuration setting is changed.

Config as Code (CaC) separates configuration from the application code. Often, configurations are stored in their own repository and managed in a different process from the primary codebase.

In this article, we explore what Config as Code means and the benefits of version control for your configuration.

## Benefits of Config as Code

Config as Code, as outlined above, is the versioned migration of configurations between different environments. Other than adding complexity to existing build and deployment processes, what does this approach gain an organization?

- **Security**: User access separation promotes least-privilege permissions and higher levels of access auditability.
- **Traceability**: Using a separate version control repository, specific to configurations, makes it easier to trace changes and updates.
- **Manageability**: Build and deployment processes specific to configurations can include additional approval, validation, and testing steps to ensure non-breaking changes.

There are many benefits when moving configuration management into its own unique process. Though complexity is introduced, and some setup and configuration is needed to remove configuration from within a codebase, the long term benefits are worth the effort.

Taking the time to plan a Config as Code approach results in a successful and managed configuration deployment process. This includes ensuring the following: 
 
- The configuration is versioned.
- It’s accessible to only those who need it.
- Secrets are encrypted.
- Proper approval gates are in place.

## Infrastructure as Code vs. Config as Code

Infrastructure as Code (IaC) concerns itself with the deployment of the underlying infrastructure necessary to support an application’s environment. This can cause confusion as it looks similar to managing configurations. 

Config as Code is about managing the specific application configuration settings themselves, which is separated from your infrastructure code and managed in its own unique process.

Infrastructure as Code and Config as Code still complement each other though and are often used hand in hand. For example, certain configurations can, and should, be managed in the configuration process that will later be consumed by the infrastructure processes. By using each approach in this way, an organization can quickly get a handle on complex configurations and how they're managed.

## Practical use of Config as Code

What does using Config as Code in practice actually imply? There are several ways to implement it in practice, and not all of them make sense for every organization. Read how the broad strokes outlined below may fit your unique needs:

- Using unique configuration source control repositories.
- Developing a tailored build and deployment process.
- Creating configuration specific test environments.
- Ensuring approval and quality assurance processes exist.
- Secrets management within configurations.

### Using unique configuration source control repositories

Typically, application configuration is packaged with the same code that is deployed. When a configuration value is changed, the entire codebase must then be deployed to affect the change. Although this means there is a versioned change, the configuration settings are normally stored in a flat file, and it can be difficult to trace the exact change made, especially in a larger code commit.

By storing and versioning configuration in its own repository, you gain:

- The benefit of independent configuration changes.
- A tailored deployment process.
- Ease of change audits. 
- Additional security controls, which are usually used for application secrets that a developer may not need access to.

### Developing a tailored build and deployment process

The deployment process for an application may not fit the needs of simple configuration changes. Often the configuration build and deployment process is simplified. The validation and testing of a configuration change may not touch as many different aspects of the application that a code change may.

With a specific configuration deployment process, additional validation steps can be built into the process. This can eliminate many process steps that don't make sense for configuration specific deployments. For example, configuration items may need to be validated, such as format and substance, which can then be handled in a configuration specific build and deployment process.

### Creating configuration specific test environments

Perhaps spinning up a full application code testing environment is not needed for a simple configuration change. Scoping a test environment just to the needs of the configuration deployment process can save time and money for an organization.

This may also mean that independent changes can happen at the same time. Application developers can test their code while a configuration change is tested as well. With this ability for parallel testing, you gain efficiency in the operation and management of the environment.

### Ensuring approval and quality assurance processes exist

Perhaps changing an API key or endpoint URI will need a different team’s input and sign-off. This approval process wouldn't typically exist if the configurations themselves were buried within the application codebase. By dovetailing the tailored build and deployment process, both approvals and quality assurance can be integrated to ensure that just the right configurations are deployed where they should be.

### Secrets management within configurations

Most configurations will include secrets necessary to access databases, key stores, or file locations that should not be widely shared. With configurations that live solely in a flat file stored next to application code, it can be difficult to secure those secrets to only those who should have access. With Config as Code, you can separate these protected secrets to encrypted values that are subject to different approval and verification processes. This increases the security of your applications and greatly decreases the chance of code or infrastructure breaches.

## Conclusion

Though there are many different ways to integrate DevOps and all of its associated processes into your environment, Config as Code makes a great starting point for a more managed and controlled application environment.

The gain in security, auditibility, manageability, and control afforded to organizations integrating Config as Code into their workflows make managing complex configurations easier and more secure than with the typical approach to bundling configuration within an application’s codebase.

!include <octopus-cac-deep-dive-video>

---

Adam Bertram is a 20+ year veteran of IT and an experienced online business professional. He’s a consultant, Microsoft MVP, blogger, trainer, published author and content marketer for multiple technology companies. Catch up on Adam’s articles at [adamtheautomator.com](http://adamtheautomator.com/), connect on [LinkedIn](https://www.linkedin.com/in/adbertram), or follow him on Twitter at [@adbertram](https://twitter.com/adbertram).
