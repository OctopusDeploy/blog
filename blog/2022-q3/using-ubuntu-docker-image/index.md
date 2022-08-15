---
title: Using the Ubuntu Docker image
description: Learn how to create custom Docker images based on the official Ubuntu base image.
author: matthew.casperson@octopus.com
visibility: public
published: 2022-08-09-1400
metaImage: blogimage-gettingstartedcontainerisation-2022.png
bannerImage: blogimage-gettingstartedcontainerisation-2022.png
bannerImageAlt: Man sitting on top of container with green circle with a power up icon
isFeatured: false
tags: 
  - Containers
  - Docker
---

The official Ubuntu Docker image is the most downloaded image from Docker Hub. With over one billion downloads, Ubuntu has proven itself to be a popular and reliable base image on which to build your own custom Docker images.

In this post I'll show you how to make the most of the base Ubuntu images while building your own Docker images.

## An example Dockerfile

This is an example `Dockerfile` with the tweaks discussed in this post. We'll go through each of the settings in subsequent sections to explain what value they add:

```Dockerfile
FROM ubuntu:22.04
RUN echo 'APT::Install-Suggests "0";' >> /etc/apt/apt.conf.d/00-docker
RUN echo 'APT::Install-Recommends "0";' >> /etc/apt/apt.conf.d/00-docker
RUN DEBIAN_FRONTEND=noninteractive \
  apt-get update \
  && apt-get install -y python3 \
  && rm -rf /var/lib/apt/lists/*
RUN useradd -ms /bin/bash apprunner
USER apprunner
```

Build the image with the command:

```bash
docker build . -t myubuntu
```

Now that we have seen how to build a custom image from the Ubuntu base image, let's go through each of the settings to understand why they were added.

## Selecting a base image

Docker images are provided for all versions of Ubuntu, including Long Term Support (LTS) releases such as 20.04 and 22.04, and normal releases like 19.04, 19.10, 21.04, and 21.10.

LTS releases are supported for 5 years, and the associated Docker images are also maintained by Canonical during this period, as described on the [Ubuntu release cycle page](https://ubuntu.com/about/release-cycle):

> These images are also kept up to date, with the publication of rolled up security updated images on a regular cadence, and you should automate your use of the latest images to ensure consistent security coverage for your users.

When creating Docker images hosting production software, it makes sense to base your images from the latest LTS release. This allows DevOps teams to rebuild their custom images on top of the latest LTS base image, which automatically includes all updates, but is also not likely to include the kind of breaking changes that can be introduced between major operating system versions.

We have used the Ubuntu 22.04 LTS Docker image as the base for our image:

```Dockerfile
FROM ubuntu:22.04
```

## Not installing suggested or recommended dependencies

Some packages have a list of suggested or recommended dependencies that are not required but are installed by default. These additional dependencies can add to the size of the final Docker image unnecessarily, as noted in [this blog post in the Ubuntu website](https://ubuntu.com/blog/we-reduced-our-docker-images-by-60-with-no-install-recommends). 

To disable the installation of these optional dependencies for all invocations of `apt-get`, the configuration file at `/etc/apt/apt.conf.d/00-docker` is created with the following settings:

```Dockerfile
RUN echo 'APT::Install-Suggests "0";' >> /etc/apt/apt.conf.d/00-docker
RUN echo 'APT::Install-Recommends "0";' >> /etc/apt/apt.conf.d/00-docker
```

## Installing additional packages

Most custom images based on Ubuntu require additional packages to be installed. For example, to run custom applications written in Python, PHP, Java, Node.js, or DotNET, our custom image must have the packages associated with those languages installed.

On a typical workstation or server, packages are installed with a simple command like:

```bash
apt-get install python3
```

The process of installing new software in a Docker image is non-interactive, which means we do not have an opportunity to respond to prompts. This means we must add the `-y` argument to automatically answer "yes" to the prompt asking to continue with the package installation:

```Dockerfile
RUN apt-get install -y python3
```

## Preventing prompt errors during package installation

The installation of some packages attempts to open additional prompts to further customize installation options. In an non-interactive environment, such as during the construction of a Docker image, attempts to open these dialogs results in errors like:

```
unable to initialize frontend: Dialog
```

These errors can be ignored as they do not prevent the packages from being installed. But the errors can be prevented by setting the `DEBIAN_FRONTEND` environment variable to `noninteractive`:

```Dockerfile
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y python3
```

The Docker website provides [official guidance on the use of the DEBIAN_FRONTEND environment variable](https://docs.docker.com/engine/faq/#why-is-debian_frontendnoninteractive-discouraged-in-dockerfiles). They consider it a cosmetic change, and recommend against permanently setting the environment variable. The command above sets the environment variable for the duration of the single `apt-get` command, meaning any subsequent calls to `apt-get` will not not have the `DEBIAN_FRONTEND` defined.

## Cleaning up package lists

Before any packages can be installed, the package list must be updated by calling:

```Dockerfile
RUN apt-get update
```

However, the package list is of little value once the required packages have been installed. It is best practice to remove any unnecessary files from a Docker image to ensure the resulting image is as small as it can be. To clean up the package list once the required packages have been installed, the files under `/var/lib/apt/lists/` are deleted.

Here we update the package list, install the required packages, and clean up the package list as part of a single command, broken up over multiple lines with a backslash at the end of each line:

```Dockerfile
RUN DEBIAN_FRONTEND=noninteractive \
  apt-get update \
  && apt-get install -y python3 \
  && rm -rf /var/lib/apt/lists/*
```

## Run as non-root user

By default the root user is run by default in a Docker container. The root user typically has far more privileges than are required when running a custom application, and so creating a new user without root privileges provides better security.

The `useradd` command [provides a non-interactive way to create new users](https://manpages.ubuntu.com/manpages/jammy/en/man8/useradd.8.html). This is not be be confused with the `adduser` command, which is a [higher level wrapper](https://manpages.ubuntu.com/manpages/jammy/en/man8/adduser.8.html) over `useradd`.

Once all configuration files have been edited and packages have been installed, we create a new user called `apprunner`:

```Dockerfile
RUN useradd -ms /bin/bash apprunner
```

This user is then set as the default user for any further options:

```Dockerfile
USER apprunner
```

## Conclusion

It is possible to use the base Ubuntu Docker images with little further customization beyond installing and required additional packages. But with a few tweaks to limit optional packages from being installed, cleaning up package lists once the packages are installed, and creating new users with limited permissions to run custom applications, we can create smaller and more secure images for our custom applications.

## Resources

* [Octopus trial](https://octopus.com/start)
* [NGINX Docker Image](https://hub.docker.com/_/ubuntu)
* [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)

## Learn more

If you're looking to build and deploy containerized applications to AWS platforms such as EKS and ECS, the [Octopus Workflow Builder](https://octopusworkflowbuilder.octopus.com/#/) populates a GitHub repository with a sample application built with GitHub Actions workflows and configures a hosted Octopus instance with sample deployment projects demonstrating best practices such as vulnerability scanning and Infrastructure as Code (IaC). 

Happy deployments! 