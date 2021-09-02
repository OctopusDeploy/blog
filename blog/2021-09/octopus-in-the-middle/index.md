---
title: "How Octopus Deploy makes for easier deployments"
description: Understand how Octopus Deploy can be the central orchestrator for a range of cloud services
author: terence.wong@octopus.com
visibility: public
published: 2021-09-01-1400
bannerImage: 
metaImage:
bannerImageAlt: 
isFeatured: false
tags:
- Product
---

In a deployment flow, there are several options. Commonly there is a build server that pushes images to a central repository. The image can then be deployed to a cloud service. In this blog, we demonstrate how Octopus can work with a range of build servers, cloud repositories and cloud targets.

![Flow Diagram](flow-diagram.png "width=500")

## Build Servers

The role of a build server is to take raw code, build it, and package it into a form ready for deployment. A common way for a build server to do this is through yaml configuration files. For the demonstration, we are going to build and push a sample web application called Random Quotes

### Travis CI

![TravisCI Success](travisci-success.png "width=500")

### Github actions

![Github Actions Success](github-actions-success.png "width=500")

### Circle CI

![CircleCI Success](circleci-success.png "width=500")


## Repositories

### Docker

![Docker Hub](dockerhub.png "width=500")

#### Google Container Registry

![GCR](gcr.png "width=500")

### Octopus Deploy


### Built-in repository

![Built-in Repository](built-in-repository.png "width=500")




## Cloud targets

### Azure

![Azure Release](azure-release.png "width=500")

![Azure Webapp](azure-webapp.png "width=500")

![Azure Website](azure-site.png "width=500")

### Google Cloud Platform


![Google Cloud Release](google-release.png "width=500")

![GCP Webapp](gcp-webapp.png "width=500")

![GCP Website](google-site.png "width=500")

### Amazon Web Services


![Amazon Release](amazon-release.png "width=500")

![Amazon Static](amazon-static.png "width=500")

http://terence-test-s3.s3-website.us-east-2.amazonaws.com/web/index.html


