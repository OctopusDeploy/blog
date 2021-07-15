---
title: Integrated Docker registry
description: Learn how Octopus is evolving to be the best tool for managing Octopus.
author: matthew.casperson@octopus.com
visibility: private # DO NOT CHANGE THIS!!!! This is not a public post!
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

The use of containers is a clear trend in our industry. The [RedHat 2021 State of Enterprise Open Source report](https://www.dropbox.com/s/z12oem8a29lv1zc/rh-enterprise-open-source-report-f27565-202101-en.pdf?dl=0) states:

> Container adoption is already widespread; just under 50% of respondents worldwide use containers in production to at least some degree. 

![](report.png)

Octopus has similarly seen steadily increasing usage of docker feeds:

![](dockerfeeds.png)

However, containers are typically stored in a container repository, which presents a challenge today as this requires an additional platform to be operated alongside Octopus.

The post proposes an integrated container repository within Octopus itself.


## What problems are we trying to solve?

The two problems this RFC aims to solve are the overheads of maintaining an external Docker registry, and the limitations with externalizing all environment specific configuration from a Docker image.

### Hosting Docker images directly

Deploying any container based application today requires orchestrating Docker image builds pushed to an external Docker registry, with the resulting images consumed by Octopus and passed along to the destination target. This means even the most simple of deployments involving Docker images requires three separate platforms to be configured correctly.

Providing a Docker registry has been suggested via [UserVoice](https://octopusdeploy.uservoice.com/forums/170787-general/suggestions/18824059-provide-private-docker-registry).

By integrating a Docker registry we remove the need for customers to implement an external registry, and provide the same kind of convenience as the current built-in feed.

### Building environment specific Docker images

The traditional advice for building environment agnostic Docker images has been to [externalize all configuration via environment variables](https://12factor.net/config). This is certainly good advice for a number of scenarios, but does require many legacy applications to be redesigned to load external values, and does not support by the traditional (and much used) configuration file modification features in Octopus.

By having Octopus build Docker images for each environment, we offer a logical upgrade path for legacy applications, especially those that have relied configuration file modification.

## Built-in Docker registry

Each space would host a built-in Docker registry implementing the [Docker HTTP API](https://docs.docker.com/registry/spec/api/). This registry would allow [OCI artifacts](https://github.com/opencontainers/artifacts) (typically Docker images, but potentially hosting any kind of OCI artifact) to be pushed and pulled by clients:

![](dockerregistry.png)

## Hosting Dockerfiles rather than images

Octopus will be expanded to host special packages that contain a `Dockerfile` file and define dependencies to additional external packages. These will be called **Source Docker Images** (SDIs).

Any step that can reference a docker image file as part of its deployment can now also reference an SDI. During package acquisition, Octopus will peform the following steps:

1. Extract the SDI along with any additional packages it references.
2. Perform file modifications with traditional features such as **Structured Configuration Variables**, **Substitute Variables in Templates**, **.NET Configuration Variables**, and **.NET Configuration Transforms**.
3. Run `docker build`, and push the resulting image to the built-in Docker registry with a deployment specific tag like `1.2.0-deployments-75`.
4. The step referencing the SDI is then passed the image reference of `octopusserver:8080/mywebapp:1.2.0-deployments-75`.

## Benefits of the new features

With so many companies embracing containers as part of their deployment strategy, extending Octopus to remove barriers to container adoption makes strategic sense.

### No need for an external Docker registry

