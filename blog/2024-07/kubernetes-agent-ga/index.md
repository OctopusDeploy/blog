---
title: "Kubernetes agent general availability"
description: The Kubernetes agent is generally available.
author: alastair.pitts@octopus.com
visibility: public
published: 
metaImage: img-blog-octopus-k8-2024.png
bannerImage: img-blog-octopus-k8-2024.png
bannerImageAlt: Octopus Deploy logo connected to a Kubernetes-branded box with an Octopus logo in it.
isFeatured: false
tags: 
  - Product
---

When we released the Kubernetes agent in Octopus Server 2024.2, we released it as an early access preview (EAP) feature. While we were very confident in the functionality and stability of the agent itself, we were also very aware of the wide variations in Kubernetes deployments and infrastructure that could impact installation, execution and stability in ways that we couldn’t test.

We have had an excellent take up of the agent during the early access period with more than 25 customers using the agent and over half of them using it for production deployments. These customers have made more than 225,000 deployments using the agent.

Since releasing the Kubernetes agent a couple of months ago, we have released 20 updates to the Kubernetes agent to improve the installation process, increase stability and also allow for more customization of the agent's Kubernetes configuration.

The Kubernetes agent will be made generally available and become our recommended way to deploy to Kubernetes in Octopus Cloud from today.

## Customer feedback

We have received excellent feedback from early adopters with many customers praising the ease of installation and use of the Kubernetes agent.

One of the benefits we found when early adopters started using the Kubernetes agent was that it dramatically improved deployment times, in some cases up to 10x faster deployments than using the Kubernetes API target. Generally, this is due to the agent not needing to perform external authentication with the target cluster.

You can read a case study with Spindlemedia [here](https://octopus.com/company/customers/casestudies/spindlemedia).

## Changes and improvements

While there have been a number of smaller bug-fixes and changes made during the EAP process, there have been a couple of important changes I’d like to highlight.

### High Availability (HA) support

Octopus Server has the ability to run in High Availability mode, however, the initial release of the Kubernetes agent did not support connecting to High Availability clusters.

Version `1.2.0` of the Kubernetes agent added the ability to specify multiple `agent.serverCommsAddresses` in the Helm values file, allowing the agent to connect to HA clusters.

You can read more details in the [documentation](https://octopus.com/docs/infrastructure/deployment-targets/kubernetes/kubernetes-agent/ha-cluster-support).

### Tenant support

Octopus Server has the ability to scope deployment targets to specific tenants. Then, when doing a tenanted deployment, the appropriately scoped deployment target is selected.

Version `1.4.0` of the Kubernetes agent added the ability to specify the tenant or tenant tags via Helm values.

You can read more details in the [documentation](https://octopus.com/docs/infrastructure/deployment-targets/kubernetes/kubernetes-agent#configuring-the-agent-with-tenants).

### Custom Octopus Server certificates

Many customers, when self-hosting Octopus Server, have a custom SSL certificate or Certificate Authority (CA) chain for their Octopus Server instance. As part of installation, the Kubernetes agent self-registers with Octopus Server. However, when a custom certificate is found, registration fails.

Version `1.7.0` of the Kubernetes agent added support for custom certificate/CA chains via the `agent.serverCertificate` Helm value.

You can read more details in the [documentation](https://octopus.com/docs/infrastructure/deployment-targets/kubernetes/kubernetes-agent#trusting-custominternal-octopus-server-certificates).

### Image pull secrets

We received a number of reports from customers that they were being rate-limited by Docker Hub when retrieving the container images when executing deployments using the agent. While the Helm chart already had support for the `imagePullSecrets` value, these values were not being used when configuring the pods used for deployment.

Version `1.9.0` of the Kubernetes agent fixed this by passing configured `imagePullSecrets` to script pods.

## What’s next?

Now that the Kubernetes agent is GA, we are focussing on another use-case for the agent, using it as a generic worker.

Many customers have given us feedback that they like to run Octopus Tentacle workers in Kubernetes clusters to take advantage of Kubernetes cluster auto-scaling and simplicity of management.

We have begun a project to allow the Kubernetes agent to be separately installed as a Kubernetes worker, to be used with any Octopus deployment.

If you would like to provide feedback or register your interest, you can do so [here](https://roadmap.octopus.com/c/108-workers-on-kubernetes).
