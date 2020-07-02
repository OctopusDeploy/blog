---
title: From Local Build to Jenkins CI
description: In this post we look at how to build a Docker image with a central CI server
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

In the previous post we took a typical Java application and created a `Dockerfile` that took care of building the code and running the resulting JAR file. By leveraging the existing Docker images provided by tools like Maven and Java itself we created repeatable and self contained build process, with the resulting Docker image that can be executed by anyone with only Docker installed.

This is a solid foundation for our build process. However, as more developers start working on a shared code base, testing requirements expand, and the resulting packages grow in size, teams require a central, shared server to manage builds. This is the role of a Continuous Integration (CI) server.

There are many CI servers available. One of the most popular is Jenkins, which is free and open source. In this blog post we'll learn how to configure Jenkins to build and publish our Docker image.

## Getting started with Jenkins

The easiest way to get started with Jenkins is to use their [Docker image](https://hub.docker.com/r/jenkins/jenkins/). Just as we created a self contained image for our own application in the previous blog post, the Jenkins Docker image provides us with the ability to launch Jenkins in a preconfigured and self contained environment with just a few commands.

To start we download the latest long term release (LTS) version of the Jenkins DOcker image with the command:

```
docker pull jenkins/jenkins:lts
```

We then launch Jenkins with the command:

```
docker run -p 8081:8080 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
```

The `-p` argument binds a port from the local workstation to a port exposed by the image. Here we use the argument `-p 8081:8080` to bind local port 8081 to the container port 8080. Note that because our own application also listens to port 8080 by default, we have chosen the next available port of 8081 for Jenkins. It is entirely up to you which local port is mapped to the container port.

The `-v` argument mounts a [Docker volume](https://docs.docker.com/storage/volumes/) to a path in the container. While a Docker container can modify data while it runs, it is best to assume that you will not be able to retain those changes. For example, each time you call `docker run` (which you may do to use an updated version of the Jenkins Docker image), a new container is created without any of the data that was modified by a previous container. Docker volumes allow us to retain modified data by exposing a persistent file system that can be shared between containers. In this example we have created a volume called `jenkins_home` and mounted it to the directory `/var/jenkins_home`. This means that all of the Jenkins data is captured in a persistent volume.

When the Docker image is run you will be presented with the log output. As part of the initial boot, Jenkins generates a random password and displays it in the logs like this:

```
*************************************************************
*************************************************************
*************************************************************

Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

4b9e47bcd9ea469687dc39f23b0adb08

This may also be found at: /var/jenkins_home/secrets/initialAdminPassword

*************************************************************
*************************************************************
*************************************************************
```