---
title: Kubernetes adoption made easy - Octopus Deploy 2018.9 
description: Kubernetes adoption made easy! Octopus 2018.9 includes our first-class support for Kubernetes Deployments including infrastructure support for Kubenetes Clusters and rich deployment steps to simplify your deployment process.
author: rob.pearson@octopus.com
visibility: private
published: 2018-10-25
metaImage: blogimage-shipping-k8s-sml.png
bannerImage: blogimage-shipping-k8s-sml.png
tags:
 - New Releases
---

![Kubernetes Adoption Made Easy - Octopus Deploy 2018.9 release banner](blogimage-shipping-k8s-sml.png "width=500")

## Kubernetes Adoption Made Easy

Octopus is proud to ship our first-class support for Kubernetes deployments! Our goal was to make Kubernetes easy for teams to adopt and migrate their projects to this popular platform. Kubernetes, or K8s, is a flexible, powerful platform for running applications and services in a reliable and scalable manner. With that power comes great complexity and it can be overwhelming and difficult to learn the configuration options and deployment YAML. We took the approach to balance power and ease-of-use to give you the best of both worlds. This took shape in the form of infrastructure support for Kubenetes Clusters and rich deployment steps to simplify your deployment process. Teams can pick the right balance for them from zero YAML configuration to full control over with `kubectl` and deployment YAML.

We believe that Kubernetes container orchestration is only the first half of the story and Octopus is the best option for the second half. Along side our first-class support, you get our best-in-class support for the following.

* Easy promotions between your environments. i.e. Dev &rarr; TEST &rarr; PROD
* Rich variable replacement
* Flexible deployment strategies
* Multi-tenant deployments

One of the best parts about our K8s deployments support is that it's complemented by our strengths with easy promotions between environments so you can  configuration variable replacement 

`TODO: Add Kubernetes Ebook Banner graphic and link here`

## Release Tour

<iframe width="560" height="315" src="https://www.youtube.com/embed/FZ8U5OuDyOw" frameborder="0" allowfullscreen></iframe>

## Kubernetes deployments made easy

Our [Kubernetes support](https://octopus.com/docs/deployment-examples/kubernetes-deployments) includes significant enhancements to Octopus infrastructure and deployment processes. 

### Infrastructure

![Kubernetes deployment targets](k8s-clusters.png "width=500")

Octopus now supports adding Kubernetes Clusters as deployment targets and all the associated configuration options. We also include support for Helm chart feeds for teams using Helm.

### Deployment Process

![Kubernetes deployment steps](k8s-steps.png "width=500")

Octopus ships with numerous new deployment steps enabling teams to deploy Docker containers to Kubernetes, execute scripts directly with `kubectl` and perform Helm upgrades.

## Offline Drop Artifacts

[Offline Package Drop targets](https://octopus.com/docs/infrastructure/offline-package-drop) can now be configured to persist the bundle as an Octopus Artifact.

Offline Package Drop targets could previously only persist the bundle to a file-system directory, which wasn't suitable for Octopus Cloud instances. Artifacts are a perfect fit for this; the deployment bundle is persisted a zip file stored against the deployment in Octopus.

## Updated Cloud SDKs

We've updated all the Cloud dependencies that ship with Octopus:

* Azure PowerShell modules upgraded from `5.7.0` to `6.8.1`. This update fixes some known issues with the `5.7.0` release of Azure PowerShell.
* Azure CLI upgraded from `2.0.42` to `2.0.45`
* AWS PowerShell modules upgraded from `3.3.225.1` to `3.3.343.0`
* AWS CLI upgraded from `1.16.6` to `1.16.15`
* Terraform CLI upgraded from `0.11.5` to `0.11.81`
* Terraform AzureRm plugin version `1.16.0`
* Terraform AWS plugin version `1.39.0`

## Breaking Changes

This release includes a major bump of Azure Powershell  modules to `6.8.1` to fix a [known issue](https://github.com/OctopusDeploy/Issues/issues/4574) with the previous `5.7.0` bundled version. Please see [Azure PowerShell release notes](https://docs.microsoft.com/en-us/powershell/azure/release-notes-azureps?view=azurermps-6.11.0) for more information.

## Upgrading

As usual [steps for upgrading Octopus Deploy](https://octopus.com/docs/administration/upgrading) apply. Please see the [release notes](https://octopus.com/downloads/compare?to=2018.9.0) for further information.

### Octopus Cloud

Octopus Cloud users will start receiving the latest bits next week during their maintenance window.

### Self-hosted Octopus

Self-Hosted Octopus customers can [download](https://octopus.com/downloads/2018.9.0) the latest release now.

!toc

## Wrap Up

That’s it for this month. Feel free to leave us a comment and let us know what you think! Go forth and deploy!