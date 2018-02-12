---
title: "Kubernetes RFC"
description: "We are designing Kubernetes support for Octopus, and we would love to know what you think."
author: michael.richardson@octopus.com
visibility: private
tags:
 - RFC 
---

Kubernetes has won the container-orchestration wars (at least for this week). Perhaps unsurprisingly, [Kubernetes is now #7 with a bullet](https://octopusdeploy.uservoice.com/forums/170787-general/suggestions/17930755-support-for-kubernetes) on the list of our top Uservoice suggestions.  

We've been thinking about what Kubernetes support in Octopus might look like, and we'd love to hear your thoughts.  Often when we are designing features and we want to know what a typical user looks like, we need only to look in the mirror. With Kubernetes, this isn't the case.  We don't currently use Kubernetes internally (though that will change as we build our hosted product), so we definitely need your help with this one! 

Our current thinking is that our Kubernetes support would take the following shape:
- A new Kubernetes Cluster target
- Two new deployment steps: Kubernetes Apply and Kubectl Script

## Kubernetes Cluster Target

We will introduce a new _Kubernetes Cluster_ target type, to represent the cluster our Kubernetes steps will execute against. 

![Kubernetes Cluster Target Option](kubernetes-cluster-target-option.png "width=500")

The target will allow you to configure the URL of the Kubernetes Cluster and the authentication details.

We will probably support the following authentication methods:

- Username + password
- Certificate
- API Token

![Kubernetes Cluster Target Details](kubernetes-cluster-target.png "width=500")

## Kubernetes Apply Step

Kubernetes supports both [declarative and imperative modes of object management](https://kubernetes.io/docs/concepts/overview/object-management-kubectl/overview/#management-techniques). 

For Octopus, it seems a natural fit to support the declarative approach.  This is implemented via the [Kubernetes Apply command](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply). We will expose this via a dedicated step.

![Kubernetes Apply Step](kubernetes-apply-step.png "width=500")

The Apply command accepts a template (JSON or YAML). This is conceptually similar to how the AWS CloudFormation or Azure Resource Group steps work in Octopus. The Kubernetes template can be sourced from a package or configured directly in the Octopus UI.     

### Container Images 

The k8s templates specify container images. For example, the template below specifies version 1.7.9 of the nginx image.   

```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  minReadySeconds: 5
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

When creating a release of a project which contains Kubernetes Apply steps, we will allow you to specify the version of the container images you wish to use.

When you deploy the release, we will substitute the correct version of the container images into your template before sending it to the Kubernetes cluster.

This is the Octopus special-sauce. It allows you snapshot a specific combination of container image versions, and progress these through your environments.

![Create Release with Container Images](kubernetes-create-release.png "width=500")

You can see in the UI-mock above that you are selecting two versions:
- The version of the package which contains your Kubernetes template
- The version of the container image _within_ the template (in this case `nginx`) 

### Variable Substitution

_blah blah blah_

## kubectl Script Step

There are many other [Kubernetes commands](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands) you may wish to execute, other than Apply.  For example: [deleting resources](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete), [scaling](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#scale), etc.

We will enable these by adding a new flavor of a Run a Script step: _Run a kubectl Script_. 

This step will allow to write your own scripts, and we ensure the `kubectl` command line is available and authenticated against the Kubernetes cluster the step is targetting.  This is conceptually similiar to our _Run an AWS CLI Script_ or the  _Run an Azure PowerShell Script_ steps, which authenticate against and provide the SDK for AWS and Azure respectively. 

## Feedback

We think this would fit nicely with the existing Octopus concepts and architecture.  But we need you to tell us if this matches the way you would expect to interact with Kubernetes. 

If you currently use Kubernetes (or are planning to), we would love to hear about your scenario and how this would fit.  Alternatively