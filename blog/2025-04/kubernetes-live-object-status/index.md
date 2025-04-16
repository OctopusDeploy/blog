---
title: Introducing Kubernetes Live Object Status
description: The Octopus Kubernetes monitor is the next major expansion of capabilities of the Octopus Kubernetes agent.
author: nick.josevski@octopus.com
visibility: public
published: 2025-04-16-1400
metaImage: kubernetes-live-object-status-2024-1500x800.png
bannerImage: kubernetes-live-object-status-2024-1500x800.png
bannerImageAlt: Stylized image of Kubernetes Live Object status in the Octopus UI showing a healthy status.
isFeatured: false
tags: 
  - Product
  - Kubernetes
  - Continuous Delivery
---

Kubernetes is rapidly becoming the dominant platform for hosting and running applications. 

At Octopus, we want to provide the best experience for deploying and monitoring your applications on Kubernetes.

To make your deployments to Kubernetes simpler, faster, and safer, Octopus has a deployment target called the Kubernetes agent. The Kubernetes agent is a small, lightweight application you install into your Kubernetes cluster. 

Kubernetes Live Object Stataus—our Kubernetes monitor—is the next major expansion of this deployment agent's capabilities. 

## Post-deployment monitoring

Monitoring is an important part of the deployment process and is even more important for Kubernetes. It gives you confidence that your applications are running as expected. 

When troubleshooting an unhealthy app, you often need to use a combination of tools and login credentials to figure out what's going wrong. That can be quite fiddly, especially with Kubernetes. Your Octopus Deploy instance already has these credentials and awareness of your applications, similar to runbooks. We're aiming to make Octopus the first port of call as you and your team continuously deliver software. 

We roll up the combined status for the objects in a given release, per environment, into a single live status.

![Project view with live status](klos-project-view.png "width=500")*Project view with live status*

## If you're already using the Kubernetes agent

If you already use the Kubernetes agent, your upgrade path will be simple.

### Upgrading your agents to the version containing the monitor

We're working on a one-click upgrade process you can access in Octopus Deploy. 

If you can’t wait until then, you can upgrade existing Kubernetes agents by running a Helm command on your cluster. [See our documentation for all the details](https://octopus.com/docs/kubernetes/live-object-status/installation#upgrading-an-existing-kubernetes-agent).

## New to using Octopus for Kubernetes deployments?

After you install the agent, it registers itself with Octopus Server as a new deployment target. This lets you deploy your applications and manifests into that cluster, without the need for workers, external credentials, or custom tooling. All new installations of the agent will have the monitor enabled.
 
### Installing the agent
 
The Kubernetes agent gets packaged and installed via a Helm chart. This makes managing the agent very simple and makes automated installation easy.

The Kubernetes monitoring component comes along for the ride. [See our docs for detailed instructions](https://octopus.com/docs/kubernetes/live-object-status/installation#upgrading-an-existing-kubernetes-agent).

![Kubernetes agent wizard configuration options](kubernetes-agent-wizard-config.png "width=500")*Kubernetes agent configuration options*

## New to Octopus Deploy entirely?

How exciting! Welcome to scalable, simple, and safe Kubernetes CD with Octopus.
 
Octopus is one user-friendly tool for developers to deploy, verify, and troubleshoot their apps. Platform engineers use this powerful tool to fully automate Continuous Delivery, manage configuration templates, and implement compliance, security, and auditing best practices.

We empower your teams to spend less time managing and troubleshooting Kubernetes deployments and more time shipping new features to improve your software.

Octopus models environments out-of-the-box and reduces the need for custom scripting. You define your deployment process once and reuse it for all your environments. You can go to production confidently as your process has already worked in other environments.

If you’re interested in trying it out, sign up for a [free 30-day trial](https://octopus.com/start).

### Getting started with the agent and monitor
 
The Octopus Kubernetes agent targets are a mechanism for executing Kubernetes steps and monitoring application health from inside the target Kubernetes cluster, rather than via an external API connection.

Like the Octopus Tentacle, the Kubernetes agent is a small, lightweight application that's installed into the target Kubernetes cluster.

You install the Kubernetes agent using Helm via the octopusdeploy/kubernetes-agent chart. For the complete details, see our docs about [installing the Kubernetes agent](https://octopus.com/docs/kubernetes/targets/kubernetes-agent#installing-the-kubernetes-agent).

### When can I use it?

The Kubernetes agent is available now as an Early Access Preview (EAP) in Octopus Cloud! If you don’t see the feature available, please reach out and we can fast-track your cloud instance getting this release.

Remember this is an opt-in upgrade for existing Octopus agents installed on your cluster(s). [See this documentation page for all the details](https://octopus.com/docs/kubernetes/live-object-status/installation#upgrading-an-existing-kubernetes-agent).

![Kubernetes agent as deployment targets](kubernetes-agent-wizard-config.png "width=500")*Kubernetes agent as deployment targets*

## How we built Kubernetes Live Object Status

To facilitate a potentially large flow of new data coming to Octopus Server, a separate and non-disruptive web host runs alongside the main host. This isolation level gives us confidence that this is an additive feature and if there are performance complications, they'll get isolated and managed with minimal impact on Octopus Server's regular operations.

The cluster-based monitoring capability uses two values to identify the incoming request: 

- The client certificate thumbprint
- An installation ID in the request headers 

Octopus Server uses a long-lived bearer token as a shared secret for authentication. The token gets generated when the monitoring capability installs in the cluster and registers with Octopus Server. This token is rotatable by customers and only valid for use on the gRPC endpoint.

This allowed us to build gRPC services to handle the data flowing from the monitoring agent in the Kubernetes clusters. [gRPC](https://grpc.io/) is a modern open-source high-performance remote procedure call (RPC) framework. This is the first time we’re using gRPC as part of an Octopus feature.

In the cluster, alongside the Octopus Kubernetes agent, we have this new component that's responsible for the monitoring aspect. It sits in the cluster and monitors the deployed resources, pumping relevant live-status data back out over gRPC to Octopus Deploy.

As we also run Octopus Deploy in Kubernetes for our self-hosted customers, we have a new nginx-based ingress configuration to help with partitioning and scalability. To find out more have a look at [how we use Kubernetes for Octopus Cloud](https://www.youtube.com/watch?v=DH7YDySEPHU).
 
### Written in Go

This is the first large-scale feature our team has built in [Golang](https://go.dev/) in Octopus. This has given us access to a large set of great libraries built for Kubernetes. Examples include Helm packages and the Argo GitOps engine. Our team got the expertise uplift from the Codefresh engineers who are now part of Octopus.

The GitOps engine is a flexible library with enough configuration and extension points for us to save very specific information on a per-resource basis. This helps us get the right information out of the cluster and back to Octopus. Go is also the de facto programming language for Kubernetes.

We're exploring options to open-source parts of our implementation. Stay tuned for when that’s all decided, as we'll have a follow-up blog post. The likely first step will be making the source available for inspection. This is part of offering more transparency into the tools we’re asking customers to run in the security context of their clusters.
 
## What's coming next

Today’s release is the EAP. The list below represents capabilities we think are worth adding next (though it's not the complete list). If you have thoughts and opinions, please reach out to us in the comments section below or on our [community Slack](https://octopus.com/slack). 

 - Terraform-based setup
 - Support Kubernetes API targets 
 - Self-hosting (this feature is only available on Octopus Cloud)
 - Octopus HA (multi-node server) support
 - Custom health checks
 - Orphan and drift detection

### This looks cool, but what if I don’t deploy to Kubernetes?

Currently, there are no plans to extend this beyond Kubernetes deployments. Please let us know where and why you'd like to use this monitoring capability.

## Let us know your thoughts

We're excited to see how you use this monitoring feature. Please let us know in the comments section below or on our [community Slack](https://octopus.com/slack) what new opportunities this opens up for your application delivery objectives.

Happy deployments!