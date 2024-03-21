---
title: External feed triggers
description: 
author: mark.coafield@octopus.com
visibility: public
published: 2024-03-25-1400
metaImage: blogimage-releasetriggersforcontainerimagesandhelmcharts-2024-1500x800-v2.png
bannerImage: blogimage-releasetriggersforcontainerimagesandhelmcharts-2024-1500x800-v2.png
bannerImageAlt: Docker and Helm logos connected to Octopus logo
isFeatured: false
tags: 
  - Product
  - Platform Engineering
  - Kubernetes
  - Continuous Integration
---

Octopus Deploy is introducing external feed triggers. This means you can trigger releases when new container images or Helm charts get pushed to their respective repositories. This new pull-based semantic lets you better automate GitOps flows in your Continuous Delivery pipelines.

## GitOps approach to Continuous Delivery

[GitOps](https://opengitops.dev/) principles provide a set of guidelines that use a declarative definition of your desired application. This gets brought into alignment with the actual state of your infrastructure.

Traditionally, Continuous Delivery workflows followed a ‘one and done’ approach to deploying an application. An artifact gets built through a Continuous Integration (CI) pipeline. This then gets pushed to a Continuous Deployment (CD) tool that performs the deployment. The CD tool is passive in this process. It sits and waits until it's notified by the build server to perform a release, using specified application dependencies.

Using a GitOps approach to Continuous Delivery, the desired state instead gets ‘pulled’ from the source. The CD server is actively comparing the system state in the target (typically Kubernetes clusters) with the desired state described by the declarative manifests in the source repository (typically Git). 

This decoupling creates a release process that more naturally supports a continuous reconciliation cycle. The changes to the desired state are automatically applied to the actual state, and drifts in the actual state get readjusted to bring them back in line with what's expected.

### A pull model for containerized applications

There are a few ways to work with a pull model for containerized applications.

As explained in [this CodeFresh article](https://codefresh.io/learn/gitops/gitops-workflow-vs-traditional-workflow-what-is-the-difference/),  one approach is to configure a deployment automator to find changes to an image repository. This then updates a YAML file in the configuration repository. This change gets identified by a GitOps agent that pulls the change and updates the cluster to match.

Or, by using Octopus for your deployments, you can template your manifests so the container image gets injected when a release gets created. Our support for [Kustomize](https://octopus.com/docs/deployments/kubernetes/kustomize) is an example of this pattern. This process puts the control for managing your containers and environment configuration outside your manifest repository. This reduces drift, merge-conflict, and security risks when using Git for environmental progression – something it was never built for.

#### External feed triggers

By configuring your Octopus project with container dependencies, you can now create external feed triggers that watch those repositories for new packages pushed by your build tool. Based on tags and [version rules](https://octopus.com/docs/releases/channels#version-rules), it detects if an image appears that is later than the image used in your previous release. Octopus then automatically creates a new release with all the latest container images or Helm chart dependencies. 

Your existing lifecycle will then promote that release through your environments or tenants, just like it does currently. If your lifecycle uses automatic release progression, then you've just set up a Continuous Delivery pipeline without explicitly letting Octopus know about your application changes! 

The details of these container images and Helm charts are already known in Octopus. This means we can use the registry locations, image names, chart names, and credentials to do this monitoring, without adding or maintaining this information anywhere else.

It all just works.

## Conclusion

Using the new external release triggers in Octopus lets you cut down the amount of configuration in your build pipeline using a GitOps-style pull model. As you're likely making more frequent changes to your application than your infrastructure manifests, having Octopus observe and react to changes in your artifact repositories reduces risks from updating those manifest repositories directly. Release triggers also let you benefit from these capabilities, even if you aren't deploying to Kubernetes.

Your CI system can focus on building and testing artifacts, and your CD system can focus on deployments and environments.

We hope the external feed triggers help you ship your application changes to your environments in a faster, safer, and simpler way than ever before.

Happy deployments!