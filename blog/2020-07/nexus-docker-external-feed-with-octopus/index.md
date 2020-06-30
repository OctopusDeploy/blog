---
title: Using a Nexus Docker registry with Octopus Deploy
description: Learn how to connect a Nexus Docker registry as an external feed to Octopus Deploy
author: shawn.sesna@octopus.com
visibility: private
published: 2021-07-01
metaImage: 
bannerImage: 
tags:
 - 
---

Connecting [Docker Hub](https://hub.docker.com) to Octopus Deploy is pretty straight forword, however, not everyone wants to use a publically avaiable Docker registry.  In these cases, you can use repository software such as [JFrog Artifactory](https://jfrog.com/artifactory) or [Sonatype Nexus](https://www.sonatype.com/product-nexus-repository).  Both JFrog and Sonatype have Open-Source Software (OSS) versions, however, JFrog does not include the Docker registry repository type in their OSS flavor.  In this post, I will demonstrate how to create a Docker registry in Nexus and connect it to Octopus Deploy.

## Nexus
The paid version includes features such as High Availability (HA), dynamic storage, and better authentication integration, but the overall operational features are all turned on in the OSS version.

### Creating a Docker registry
Creating a Docker registry repository in Nexus is pretty easy.  Once logged in, navigate to the **Server Administration** tab (gear icon).

![](nexus-server-administration.png)

From here, click on the **Repositories** tab on the left hand side, then click **Create repository**

![](nexus-create-repository.png)

Choose the **docker (hosted)** repository type

![](nexus-docker-hosted.png)

This part of the process is where it differs from the other repository types such as NuGet or Maven2.  Whereas the other two would use https://ServerUrl/repository/RepositoryName, Nexus Docker registries require a port assignment.  In this example, I have used port 8443, this will become important later when we tag our images.

![](nexus-docker-repo.png)

The rest of the options can be left at default, click the **Create repository** button at the bottom of the page when you're done.

![](nexus-docker-repo-created.png)

### Tagging images
In order to be able to upload a docker image to our Nexus Docker registry, we have to tag the image appropriately.  This is accomplished by using the following tag scheme: ServerUrl:Port/ImageName:Version.  For example,
```
nexus.octopusdemos.app:8443/pitstop-auditlogservice:1.0.0.0
```
Note the repository name of `nexus-docker` is not present in the tag.  This is where the port designation is important as it identifies which registry this will belong to when uploaded to Nexus.

## Connecting to Octopus Deploy
When connecting other repository types to Octopus Deploy, you simply provide the url from the Copy Url button within Nexus

![](nexus-copy-url.png)

While the button is present for our Docker registry, this is not how we connect to the repository with Octopus Deploy

:::warning
Using the URL from this page when creating an External Feed within Octopus Deploy and testing it will produce a false positive.  While the endpoint will respond with a success message and even provide search results, it will fail when you attempt to use it in a deployment process.
:::

### Creating the External Feed
To create an External Feed in Octopus Deploy click on the **Library** tab, then **External Feeds**

![](octopus-external-feed.png)

Once on the External Feeds screen, click the **ADD FEED** button

![](octopus-add-external-feed.png)

Enter the following information for the External Feed:
- Feed type: Docker Container Registry
- Name: Give it a descriptive name
- URL: https://ServerUrl:Port (e.g. https://nexus.octopusdemos.app:8443)
- Registry Path: leave blank
- Credentials: Enter credentials if they are required

![](octopus-docker-external-feed.png)

Click **SAVE AND TEST** when done.

Enter an image name (partial names are supported) and click **SEARCH** to verify the connection is working properly

![](octopus-test-feed.png)

## Conclusion
Nexus has a unique way of handling Docker registries as repository types.  In this post I demonstrated what is necessary to create a registry and connect it to Octopus Deploy.