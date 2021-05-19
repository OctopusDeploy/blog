---
title: Deploying to Google Cloud Run
description: Learn how to deploy a container to the Google Cloud Run service
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

Google Cloud Run is a relatively new Platform as a Service (PaaS) offering on the Google Cloud Platform (GCP). It provides the ability to run and scale container images while only paying for the time that a request is being processed.

In this blog post we'll look at how to deploy a sample application cloud run, and use the traffic shaping rules to perform deployment strategies like feature branches, canary, and blue/green.

## Deploying a sample application

We'll start by deploying a sample application called Random Quotes. This Java Spring web application has been pushed to Docker Hub, and we'll use the latest tag at the time of writing which is `octopussamples/randomquotesjava:0.1.189`.

The first thing to do when deploying to cloud run is to push the Docker image to a Google Container Registry (GCR). Cloud run does not support external registries.

Using the traditional `docker` CLI tool, we can pull and push an image with the following commands, making sure to replace `cloudrun-314201` with the project ID that will hold your cloud run services:

```
docker pull octopussamples/randomquotesjava:0.1.189
docker tag octopussamples/randomquotesjava:0.1.189 gcr.io/cloudrun-314201/randomquotesjava:0.1.189
docker push gcr.io/cloudrun-314201/randomquotesjava:0.1.189
```

Other tools like `skopeo` have been developed to provide more a convenient means of copying Docker images. The command below will directly copy the image from the Docker Hub registry to GCR:

```
skopeo copy docker://octopussamples/randomquotesjava:0.1.189 docker://gcr.io/cloudrun-314201/randomquotesjava:0.1.189
```

:::hint
If you try to reference a Docker image from an external registry, you will receive the error:

```
ERROR: (gcloud.beta.run.services.replace) Expected a Container Registry image path like [region.]gcr.io/repo-path[:tag and/or @digest] or an Artifact Registry image path like [region-]docker.pkg.dev/repo-path[:tag and/or @digest], but obtained octopussamples/randomquotesjava:0.1.189
```
:::

The next step is to define a service YAML resource in a file called `service.yaml`. 

If you are familiar with Kubernetes, the structure of the following YAML will look familiar. It follows the `apiVersion`, `kind`, `metadata`, and `spec` layout that all Kubernetes resources use. In fact the service we are defining here is part of [Knative](https://knative.dev/docs/reference/api/serving-api/), because cloud run is a managed implementation of the Knative service:

```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: randomquotes
spec:
  template:
    spec:
      containers:
        - image: gcr.io/cloudrun-314201/randomquotesjava:0.1.189
```

To deploy this service, run the command:

```bash
gcloud beta run services replace service.yaml
```