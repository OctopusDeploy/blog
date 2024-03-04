---
title: "Role Based Access Control demo"
description: "Create an Octopus target authenticated with a service account, as part of our Kubernetes training series."
author: matthew.casperson@octopus.com
visibility: public
published: 2022-01-01-1200
metaImage: blogimage-testingkubernetes-2022.png
bannerImage: blogimage-testingkubernetes-2022.png
bannerImageAlt: Kubernetes logo on an open laptop screen
isFeatured: false
tags: 
  - DevOps
  - Containers
  - Cloud Orchestration
  - Continuous Delivery
  - Docker 
  - Kubernetes
  - Kubernetes Training
---

This post is the 14th in our Kubernetes training series, providing DevOps engineers with an introduction to Docker, Kubernetes, and Octopus. 

This video demonstrates how to create an Octopus target that authenticates to the cluster with a service account token.

[If you don't already have Octopus account, you can start a free trial.](https://oc.to/octopus-k8s-training-trial)

<p style="text-align:center"><iframe width="560" height="315" src="https://www.youtube.com/embed/Rlhzmlt-7zs?si=4J_1JNpxPFKKNAZ1" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe></p>

You can work through the series using the links below.

!include <k8s-training-toc>

## Example code

### RBAC resources

This is the compound YAML document containing the RBAC resources used to limit an service account to a single namespace:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: octopub-deployer
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: octopub-deployer-role
rules:
- apiGroups: ["", "extensions", "apps", "networking.k8s.io"]
  resources: ["deployments", "replicasets", "pods", "services", "ingresses", "secrets", "configmaps"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: [""]
  resources: ["namespaces"]
  verbs: ["get"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: octopub-deployer-rolebinding
subjects:
- kind: ServiceAccount
  name: octopub-deployer
  apiGroup: ""
roleRef:
  kind: Role
  name: octopub-deployer-role
  apiGroup: ""
---
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: octopub-deployer-secret
  annotations:
    kubernetes.io/service-account.name: "octopub-deployer"
```

### Target creation script

This script creates a new Octopus token account and target by extracting the token from a secret and the Kubernetes URL from the current Kubernetes context:

```bash
SERVER=$(kubectl config view -o json | jq -r '.clusters[0].cluster.server')
TOKEN=$(kubectl get secret octopub-deployer-secret -n octopub -o json | jq -r '.data.token' | base64 -d)

echo "##octopus[create-tokenaccount \
  name=\"$(encode_servicemessagevalue "Octopub #{Octopus.Environment.Name}")\" \
  token=\"$(encode_servicemessagevalue "${TOKEN}")\" \
  updateIfExisting=\"$(encode_servicemessagevalue 'True')\"]"

echo "##octopus[create-kubernetestarget \
  name=\"$(encode_servicemessagevalue "Octopub #{Octopus.Environment.Name}")\" \
  octopusRoles=\"$(encode_servicemessagevalue 'Octopub')\" \
  clusterUrl=\"$(encode_servicemessagevalue "${SERVER}")\" \
  octopusAccountIdOrName=\"$(encode_servicemessagevalue "Octopub #{Octopus.Environment.Name}")\" \
  namespace=\"$(encode_servicemessagevalue "octopub")\" \
  octopusDefaultWorkerPoolIdOrName=\"$(encode_servicemessagevalue "Laptop")\" \
  updateIfExisting=\"$(encode_servicemessagevalue 'True')\" \
  skipTlsVerification=\"$(encode_servicemessagevalue 'True')\"]"
```

## Resources

* [Octopus trial](https://oc.to/octopus-k8s-training-trial)
* [Mixing Kubernetes Roles, RoleBindings, ClusterRoles, and ClusterBindings](https://octopus.com/blog/k8s-rbac-roles-and-bindings)
* [Service messages](https://octopus.com/docs/deployments/custom-scripts/logging-messages-in-scripts#service-message)

## Learn more

If you want to build and deploy containerized applications to AWS platforms such as EKS and ECS, try the [Octopus Workflow Builder](https://octopusworkflowbuilder.octopus.com/#/). The Builder populates a GitHub repository with a sample application built with GitHub Actions workflows and configures a hosted Octopus instance with sample deployment projects demonstrating best practices such as vulnerability scanning and Infrastructure as Code (IaC). 

Happy deployments! 
