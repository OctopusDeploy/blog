---
title: "How Octopus Deploy makes for easier deployments"
description: Understand how Octopus Deploy can be the central orchestrator for a range of cloud services
author: terence.wong@octopus.com
visibility: public
published: 2021-09-01-1400
bannerImage: blogimage-octopusmakesforeasierdeployments-2021.png
metaImage: blogimage-octopusmakesforeasierdeployments-2021.png
bannerImageAlt: Octopus tentacles slither out from an Octopus website window. Each tentacle holds a logo: Docker, AWS, GCP and CircleCI.
isFeatured: false
tags:
- Product
---

Deployment tools like build servers, image repositories and deployment targets come in different shapes and sizes. Octopus Deploy makes deployments easier by supporting a range of tools though a clear user interface. I show you some deployment tools and how Octopus fits with them in this blog.

## Build Servers

The role of a build server is to take raw code, build it, and package it into a form ready for deployment. This can be done through YAML files. I build and push a sample web application called Random Quotes to two container registries: Docker Hub and Google Container Registry.

The repository I used is from the Octopus Deploy [samples repository](https://github.com/OctopusSamples/RandomQuotes-JS). Each build server requires a configuration folder in the root level containing a YAML  file. The repository stores access keys to the build server and container registry. These keys are used in the YAML file to authenticate, build and push an image. I used Github Actions to push to the Google Content Registry and Travis CI and CircleCI to push to DockerHub. Build servers are interchangable and they can push to any content registry.


### Github Actions

Github Actions allows DevOps operations to be performed directly on a Github repository. Github Actions is organised into workflows, a feature that allows configuration files to specify a deployment process. The Github community maintains templates that cover different deployment targets. I used a basic one to set up a job to deploy to GCR.

![Github Actions Success](github-actions-success.png "width=500")

### Travis CI

Travis CI is an open-source continuous delivery tool. It is free to sign up and works with GitHub. I connected my Github repository to Travis CI. Travis CI automatically detects changes on the repository and triggers a build. There wasn't a Travis CI template to use, but there were resources online to make it work.

![TravisCI Success](travisci-success.png "width=500")

### Circle CI

Circle CI is also free to start and works with GitHub. Circle CI does have templates that you can import. This was helpful in quickly setting up a deployment flow.

![CircleCI Success](circleci-success.png "width=500")

## Repositories

A image repository is a place to store deployable images. Octopus Deploy uses these images and deploys them to a target. Octopus Deploy supports image repositories like Docker Hub, Google Container Registry, Azure Container Registry and AWS Elastic Container Registry.

### Docker Hub

Docker Hub is a central repository for Docker images. It is free to sign up and create public repositories.

![Docker Hub](dockerhub.png "width=500")

### Google Container Registry

Google Container Registry is a container registry for the Google Cloud Platform. I added the Google Container Registry to the Github Actions step to push to Google Container Registry directly.

![GCR](gcr.png "width=500")

### Built-in repository

Octopus Deploy contains a built-in repository to manage local packages. The built-in repository can be helpful for self-managing the packages that are deployed or keeping them private.

![Built-in Repository](built-in-repository.png "width=500")

### Octopus Deploy

We have explored different tools that are part of a deployment. Octopus Deploy is a deployment tool that takes an image and deploys it to different targets. Octopus has wide support for deployment tools and a clear UI. Octopus Deploy uses dedicated environments to split releases into stages. The environment view tells the user which environment a release is in.

![Octopus UI](octopus-ui.png "width=500")

### Deployment Targets

Octopus Deploy deploys to Azure, Google and Amazon through an Octopus step. The step uses the Random Quotes image, authenticates with the Cloud and calls the Cloud API to create a web application.

![Azure Release](azure-release.png "width=500")

The image below shows the Web Application deployed onto Azure. This is the same view for Google and Amazon. The Web Application displays a random quote by a historical figures on every refresh.

![Azure Website](azure-site.png "width=500")

## Conclusion

In this blog, I have introduced build servers, content repositories and deployment targets. Octopus Deploy makes deployments easier by supporting a range of deployment tools and providing a clean UI. [If you haven't already, you can start a free trial.](https://octopus.com/start)
