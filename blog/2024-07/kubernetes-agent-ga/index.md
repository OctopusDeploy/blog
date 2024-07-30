---
title: Kubernetes agent now generally available
description: The Kubernetes agent is generally available.
author: alastair.pitts@octopus.com
visibility: public
published: 2024-07-29-1400
metaImage: img-blog-octopus-k8-2024.png
bannerImage: img-blog-octopus-k8-2024.png
bannerImageAlt: Octopus Deploy logo connected to a Kubernetes-branded box with an Octopus logo in it.
isFeatured: false
tags: 
  - Product
  - Kubernetes
---

We released the Kubernetes agent in Octopus Server 2024.2 as an early access preview (EAP). While we were very confident in the functionality and stability of the agent, we were also aware of the variations in Kubernetes deployments and infrastructure. These variations can impact installation, execution, and stability of the agent.

We had excellent uptake of the agent during the early access period. More than 25 customers regularly deployed with the agent and over half of them used it for production deployments. These customers deployed more than 225,000 times using the agent.

We've released more than 20 updates to the Kubernetes agent since releasing it. These updates have improved the installation process, increased stability, and allowed for more customization of the agent's Kubernetes configuration.

The Kubernetes agent is now generally available and is our recommended way to deploy to Kubernetes in Octopus Cloud.

## Customer feedback

We received excellent feedback about the agent from early adopters. Many customers praised the ease of installation and use.

One benefit we found when early adopters started using the Kubernetes agent was that it dramatically improved deployment times. In some cases, customers deploy up to 10 times faster compared to using the Kubernetes API target. This is mostly because the agent doesn't perform external authentication with the target cluster.

You can learn how [Spindlemedia uses the Kubernetes agent](https://octopus.com/company/customers/casestudies/spindlemedia) to improve deployment speed.

## Changes and improvements

We made lots of small bug-fixes and changes during the EAP process. There are a couple of important changes I’d like to highlight.

### High Availability (HA) support

Octopus Server can run in High Availability mode, but the initial release of the Kubernetes agent didn't support connecting to High Availability clusters.

Version 1.2.0 of the Kubernetes agent added the ability to specify multiple `agent.serverCommsAddresses` in the Helm values file. This allows the agent to connect to HA clusters.

You can read more details in our [documentation about HA cluster support](https://octopus.com/docs/infrastructure/deployment-targets/kubernetes/kubernetes-agent/ha-cluster-support).

### Tenant support

Octopus Server can scope deployment targets to specific tenants. This means Octopus selects the appropriate deployment target for the tenant.

Version 1.4.0 of the Kubernetes agent added the ability to specify the tenant or tenant tags via Helm values.

Learn more in our [docs about configuring the agent with tenants](https://octopus.com/docs/infrastructure/deployment-targets/kubernetes/kubernetes-agent#configuring-the-agent-with-tenants).

### Custom Octopus Server certificates

Many customers who self-host Octopus Server use a custom SSL certificate or Certificate Authority (CA) chain for their Octopus Server instance. During installation, the Kubernetes agent self-registers with Octopus Server. When using a custom certificate or CA chain, registration fails.

Version 1.7.0 of the Kubernetes agent added support for custom certificate/CA chains via the `agent.serverCertificate` Helm value.

You can read more in our [documentation about certificates](https://octopus.com/docs/infrastructure/deployment-targets/kubernetes/kubernetes-agent#trusting-custominternal-octopus-server-certificates).

### Image pull secrets

Customers reported they were being rate-limited by Docker Hub when executing deployments. The Helm chart already supported the `imagePullSecrets` value, but these values weren't used when configuring the pods used for deployments.

Version 1.9.0 of the Kubernetes agent fixed this by passing configured `imagePullSecrets` to script pods.

## What’s next?

Now the Kubernetes agent is GA, we're focusing on another use-case for the agent – using it as a generic worker.

Many customers told us they'd like to run Octopus Tentacle workers in Kubernetes clusters. They want to take advantage of Kubernetes cluster auto-scaling and simplicity of management.

We've started a project to support installing the Kubernetes agent as a Kubernetes worker, so you can use it with any deployment. We'd love to hear from you if you have feedback or want to [register your interest](https://roadmap.octopus.com/c/108-workers-on-kubernetes).

Happy deployments!
