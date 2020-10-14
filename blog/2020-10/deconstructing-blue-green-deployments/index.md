---
title: Deconstructing blue/green deployments in Kubernetes
description: Learn how to manually implement blue green deployments in Kubernetes and Octopus.
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

In addition to the standard deployment strategies of recreate or rolling natively implemented by Kubernetes, Octopus exposes the ability to perform blue/green deployments. This tick box option allows individual Kubernetes deployments to be deployed in a blue/green fashion, complete with service cut over and a cleanup of old resources.

But there are times when this checkbox approach doesn't work. If you want to halt the cut over to the new deployment to allow for manual testing, orchestrate multiple deployments (for example a frontend and backend application), or use feature branches, the you need to recreate the blue/green deployment process for yourself.

In this blog post and associated screencast I'll show you how to recreate a blue/green deployment, and deploy a mock feature branch as an demonstration of the process.

## Screencast

## Creating the initial deployment

For this demo we will deploy the [httpd](https://hub.docker.com/_/httpd) docker image into Kubernetes. This gives us a web server we can point our browser and command line tools at, and we'll make use of the tags like `2.4.46-alpine` as a way of simulating feature branches.

We start by deploying a new Kubernetes deployment resource with the **Deploy Kubernetes containers** step.

The resource must have a unique name, which we create by appending the string `-#{Octopus.Deployment.Id | ToLower}` to the resource name. The `Octopus.Deployment.Id` variable is unique for each deployment, and so we can be sure that each new deployment will create a new Kubernetes resource.

We also define a label called `FeatureBranch` set to the value of a variable we'll create in the next section called `PackagePreRelease`.

The YAML below can be pasted into the **Edit YAML** section of the **Deploy Kubernetes containers** step to configure the resource:

```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: 'httpd-#{Octopus.Deployment.Id | ToLower}'
  labels:
    FeatureBranch: '#{PackagePreRelease}'
spec:
  selector:
    matchLabels:
      octopusexport: OctopusExport
  replicas: 1
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        FeatureBranch: '#{PackagePreRelease}'
        octopusexport: OctopusExport
    spec:
      containers:
        - name: httpd
          image: index.docker.io/httpd
          ports:
            - name: web
              containerPort: 80
```

## Defining the variables

We need to create two variables to capture the name of a feature branch.

The first variable is called `PackagePreRelease`, and the value is set to `#{Octopus.Action[Deploy HTTPD].Package[httpd].PackageVersion | VersionPreReleasePrefix}`. This template string extracts the version (or image tag) from the container called `httpd` in the step called `Deploy HTTPD`, and extracts the prerelease string via the `VersionPreReleasePrefix` filter.

This means that the `PackagePreRelease` variable will be empty for a mainline deployment, and set to `alpine` for what we were calling feature branch deployments in this demo.

The second variable is called `ServiceSuffix`, and the value is set to `#{if PackagePreRelease}-#{PackagePreRelease}#{/if}`. This template prepends a dash before the value of the `PackagePreRelease` if `PackagePreRelease` is not empty. Otherwise `ServiceSuffix` is left as an empty string.

This means that the `ServiceSuffix` variable will be empty for a mainline deployment, and set to `-alpine` for what we were calling feature branch deployments in this demo.