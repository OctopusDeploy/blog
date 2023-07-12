---
title: Enterprise deployment patterns reference implementation
description: Learn how to deploy the example reference implementation demonstrating the enterprise patterns.
author: matthew.casperson@octopus.com
visibility: public
published: 2023-01-06-1400
metaImage: blogimage-enterprise-series-reference-implementation-2023.png
bannerImage: blogimage-enterprise-series-reference-implementation-2023.png
bannerImageAlt: People building an unstable tower with blue blocks, beside 2 people building a stable, lower tower with blue blocks.
isFeatured: false
tags:
 - DevOps
 - Enterprise
---

It's one thing to talk about [enterprise deployment patterns](https://octopus.com/blog/enterprise-patterns) in abstract terms, but teams can only realize the value of these patterns by implementing them. To that end , we developed the [enterprise patterns reference implementation](https://github.com/OctopusSolutionsEngineering/EnterprisePatternsReferenceImplementation). It's a Docker Compose-based application stack deploying and configuring a sample Octopus instance containing practical examples of the following patterns:

- Managed space per business unit/application
- Managed instance per business unit/region
- Managed instance per environment

This post describes how to install and use the reference implementation.

## Support levels

The process presented in this post is part of a pilot program to support platform engineering teams. It incorporates tools with varying levels of official support. The Octopus support team will make reasonable endeavors to help customers that wish to use this process. However, existing support Service Level Agreements (SLAs) do not apply to many of the tools described below.

:::warning 
This post describes tools and processes we're seeking feedback on. They're not covered by existing SLAs, and you shouldn't apply them to production systems, or rely on them for production deployments. 
:::

## Tools to get started

To use the reference implementation, ensure you have the following tools installed:

* Windows users must install [WSL2](https://learn.microsoft.com/en-us/windows/wsl/install).
* [Git](https://git-scm.com/downloads), which clones the sample GitHub repository.
* [Docker and Docker Compose](https://docs.docker.com/get-docker/), which boots the application stack.
* [Terraform](https://developer.hashicorp.com/terraform/downloads), which configures Octopus.
* [minikube](https://minikube.sigs.k8s.io/docs/start/), which creates a local development Kubernetes cluster.
* [OpenSSL](https://www.openssl.org/), which parses the minikube certificates.
* [curl](https://curl.se/), which downloads files.
* [jq](https://stedolan.github.io/jq/), which parses JSON files.

## Environment variables to get started

You also need a [Docker Hub](https://hub.docker.com/) account. Octopus requires Docker Hub credentials to query image tags hosted by the service. The Docker Hub credentials are exposed by two environment variables:

* `TF_VAR_docker_username`, which holds the Docker Hub username.
* `TF_VAR_docker_password`, which holds the Docker Hub password.

Finally you need an Octopus license key. This is an XML document, which you must base 64 encode and expose in the `OCTOPUS_SERVER_BASE64_LICENSE` environment variable.

You can find more information on persisting Linux environment variables (which also applies to Windows WSL users) [in the Ubuntu help docs](https://help.ubuntu.com/community/EnvironmentVariables#Persistent_environment_variables) and for macOS [on this Stack Exchange page](https://apple.stackexchange.com/questions/356441/how-to-add-permanent-environment-variable-in-zsh).

:::hint
Octopus employees can find sample credentials in the `Sample environment vars for EnterprisePatternsReferenceImplmenetation` note in the internal password manager.
:::

## Clone the repo

Clone the sample repo with the command:

```bash
git clone https://github.com/OctopusSolutionsEngineering/EnterprisePatternsReferenceImplementation.git
```

Enter the newly-cloned repo directory with the command:

```bash
cd EnterprisePatternsReferenceImplementation
```

Then initiate the creation of the sample application stack with the command:

```bash
./initdemo.sh 
```

The first initialization will take some time as all the Docker images are downloaded and Octopus is configured via Terraform. Once the initialization is complete, the following URLs are available:

- http://localhost:18080, which grants access to Octopus. The username is `admin` and the password is `Password01!`.
- http://localhost:3000, which grants access to Gitea (a hosted Git platform). The username is `octopus` and the password is `Password01!`.

## Exploring the Octopus instance

Open `http://localhost:18080`. Log in with the credentials above, and you'll enter an Octopus instance configured with sample projects and spaces.

We configured the **Default** space as a management space. This space creates additional spaces representing regional teams and deploys template projects to them.

The **Development** and **Test/Production** spaces represent the "Managed instance per environment" pattern. This is where you refine a deployment process in the **Development** space and then promote it to the **Test/Production** space.

You'll also notice we preconfigured the **Default** space with global resources, including environments, lifecycles, accounts, certificates, targets, variable sets, tenants, and feeds. These all support the sample applications available for propagation to managed spaces.

We'll reference these resources in subsequent posts as we describe the implementation of the enterprise patterns.

## Conclusion

The reference implementation serves as the foundation of this blog series. It lets DevOps teams explore a practical demonstration of Octopus used to support complex deployments across many business units or regions.

If you're interested in orchestrating Octopus with an Infrastructure as Code (IaC) approach, you'll find a wealth of example Terraform configuration files in this reference implementation that you can easily apply to your own instances.

In the [next post](https://octopus.com/blog/managed-space-pattern), we'll cover the high-level configuration of the **Default** space supporting the "Managed space per business unit/application" and "Managed instance per business unit/region" patterns.

We're currently refining our approach to these enterprise patterns, so if you have any suggestions or feedback about the approach described here, please leave a comment on [this GitHub issue](https://github.com/OctopusSolutionsEngineering/EnterprisePatternsReferenceImplementation/issues/1).

Happy deployments!