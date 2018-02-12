---
title: "Kubernetes RFC"
description: "We are designing Kubernetes support for Octopus, and we would love to know what you think."
author: michael.richardson@octopus.com
visibility: private
tags:
 - RFC 
---

Kubernetes has won the container-orchestration wars (at least for this week). Perhaps unsurprisingly, [Kubernetes is now #7 with a bullet](https://octopusdeploy.uservoice.com/forums/170787-general/suggestions/17930755-support-for-kubernetes) on the list of our top Uservoice suggestions.  

We've been thinking about what Kubernetes support in Octopus might look like, and we'd love to hear your thoughts.  Often when we are designing features and we want to know what a typical user looks like, we need only to look in the mirror. With Kubernetes, this isn't the case.  We don't currently use Kubernetes internally (though that will change as we build our hosted product).  So we definitely need your help with this one! 

Our current thinking is that our Kubernetes support would take the following shape:
- A new Kubernetes Cluster target
- Two new deployment steps: Kubernetes Apply and Kubectl Script

## Kubernetes Cluster Target

We will introduce a new _Kubernetes Cluster_ target type, to represent the cluster our Kubernetes steps will execute against. 

**TODO: Image**

The target will allow you to configure the URL of the Kubernetes Cluster and the authentication details.

We will probably support the following authentication methods:

- Username + password
- Certificate
- API Token

**TODO: Image**

## Kubernetes Apply Step

Kubernetes supports both [declarative and imperative modes of object management](https://kubernetes.io/docs/concepts/overview/object-management-kubectl/overview/#management-techniques). 

For Octopus, it seems a natural fit to support the declarative approach.  This is implemented via the [Kubernetes Apply command](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply). We will expose this via a dedicated step.

**TODO: Image**

The Apply command accepts a template (JSON or YAML). This is conceptually similar to how the AWS CloudFormation or Azure Resource Group steps work in Octopus. The Kubernetes template can be sourced from a package or configured directly in the Octopus UI.     

### Container Images 

Intro

New Target
Kubernetes Apply
kubectl script 

transforms/variables

sign-off