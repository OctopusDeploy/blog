---
title: Enterprise Deployment Patterns Reference Implementation
description: Learn how to deploy the example reference implementation demonstrating the enterprise patterns
author: matthew.casperson@octopus.com
visibility: public
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

It is one thing to talk about [enterprise deployment patterns](/blog/2023-06/enterprise-patterns/index.md) in abstract terms, but at the end of the day teams can only realize the value of these patterns by implementing them. To that end we have developed the [enterprise patterns reference implementation](https://github.com/OctopusSolutionsEngineering/EnterprisePatternsReferenceImplementation), which is a Docker Compose based application stack deploying and configuring a sample Octopus instance containing practical examples of the following patterns:

* Managed space per business unit/application
* Managed instance per business unit/region
* Managed instance per environment

This post describes how to install and use the reference implementation.

## Prerequisite tools

To use the reference implementation, you must ensure you have a number of prerequisite tools installed:

* Windows users must install [WSL2](https://learn.microsoft.com/en-us/windows/wsl/install).
* [Git](https://git-scm.com/downloads), which clones the sample GitHub repository.
* [Docker and Docker Compose](https://docs.docker.com/get-docker/), which boots the application stack.
* [Terraform](https://developer.hashicorp.com/terraform/downloads), which configures Octopus.
* [minikube](https://minikube.sigs.k8s.io/docs/start/), which creates a local development Kubernetes cluster.
* [OpenSSL](https://www.openssl.org/), which parses the minikube certificates.
* [curl](https://curl.se/), which downloads files.
* [jq](https://stedolan.github.io/jq/), which parses JSON files.

## Prerequisite environment variables

You must also have a [Docker Hub](https://hub.docker.com/) account. Octopus requires Docker Hub credentials to query image tags hosted by the service. The Docker Hub credentials are exposed by two environment variables:

* `TF_VAR_docker_username`, which holds the Docker Hub username.
* `TF_VAR_docker_password`, which holds the Docker Hub password.

Finally you need an Octopus license key. This is an XML document, which you must base 64 encode and expose in the `OCTOPUS_SERVER_BASE64_LICENSE` environment variable.

You can find more information on persisting Linux environment variables (which also applies to Windows WSL users) [here](https://help.ubuntu.com/community/EnvironmentVariables#Persistent_environment_variables) and for macOS [here](https://apple.stackexchange.com/questions/356441/how-to-add-permanent-environment-variable-in-zsh).

:::hint
Octopus employees can find sample credentials in the `Sample environment vars for EnterprisePatternsReferenceImplmenetation` note in the internal password manager.
:::

## Clone the repo

Clone the sample repo with the command:

```bash
git clone https://github.com/OctopusSolutionsEngineering/EnterprisePatternsReferenceImplementation.git
```

Enter the newly clone repo directory with the command:

```bash
cd EnterprisePatternsReferenceImplementation
```

Then initiate the creation of the sample application stack with the command:

```bash
./initdemo.sh 
```

The first initialization will take some time as all the Docker images are downloaded and Octopus is configured via Terraform. Once the initialization is complete, the following URLs are available:

* http://localhost:18080, which grants access to Octopus. The username is `admin` and the password is `Password01!`.
* http://localhost:3000, which grants access to Gitea (a hosted Git platform). The username is `octopus` and the password is `Password01!`.

## Exploring the Octopus instance

Open http://localhost:18080, log in with the credentials listed above, and you will enter an Octopus instance configured with a number of sample projects and spaces.

The `Default` space has been configured as a management space. This space creates additional spaces representing regional teams and deploys template projects to them.

The `Development` and `Test/Production` spaces represent the "Managed instance per environment" pattern where a deployment process is refined in the `Development` space and then promoted to the `Test/Production` space.

You will also notice the `Default` space has been preconfigured with a number of global resources including environments, lifecycles, accounts, certificates, targets, variable sets, tenants, and feeds. These all support the sample applications available to be propagated to managed spaces.

We'll reference these resources in subsequent posts as we describe the implementation of the enterprise patterns.

## Conclusion

The reference implementation serves as the foundation of this blog series and allows DevOps teams to explore a practical demonstration of Octopus used to support complex deployments across many business units or regions.

Those interested in orchestrating Octopus with an Infrastructure as Code (IaC) approach will find a wealth of example Terraform configuration files in this reference implementation that is easily applied to your own instances.

In the [next post](/blog/2023-06/managed-space-pattern/index.md) we'll cover the high level configuration of the `Default` space supporting the "Managed space per business unit/application" and "Managed instance per business unit/region" patterns.

We are currently refining our approach to these enterprise patterns, so if you have any suggestions or feedback about the approach described here, please leave a comment on [this GitHub issue](https://github.com/OctopusSolutionsEngineering/EnterprisePatternsReferenceImplementation/issues/1).