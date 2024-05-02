---
title: Expanding the use of execution containers
description: Octopus 2024.1 added the ability to use an inline Dockerfile or a URL to a Git repository to build the Docker container used for a deployment.
author: henrik.andersson@octopus.com
visibility: public
published: 2024-05-06-1400
metaImage: blogimage-containerdocker.png
bannerImage: blogimage-containerdocker.png
bannerImageAlt: Man standing with a laptop in front of a large blue container.
isFeatured: false
tags: 
  - Product
  - Containers
  - Docker
  - Git
---

With the introduction of execution containers for steps, we simplified how you deploy applications using Octopus. We provided a lightweight and portable solution for bundling dependencies needed in your deployments into Docker containers.

If you can't or don't want to use the container images we provide, managing Dockerfiles and CI pipelines to publish the Docker containers and ensuring smooth deployment workflows can still present challenges. 

We're happy to announce we improved this feature in 2024.1 with new options that simplify this process. You now have the flexibility to choose to use an inline Dockerfile or a URL to a Git repository to build the Docker container image for a deployment.

## Challenges using execution containers

Before jumping into the new options, let's discuss the challenges of creating Docker container images for deploying applications with execution containers. 

Creating a Docker container image for use as an execution container involves writing a Dockerfile. This specifies the steps to build the container image with the necessary dependencies, and a CI/CD pipeline to publish the container image to a registry that Octopus can access (like DockerHub or Azure Container Registry).

While Dockerfiles are powerful and customizable, managing them across different projects can become cumbersome. This is especially the case when dealing with many dependencies or complex deployment processes.

Any friction in this process can lead to delays in delivering updates and features to your end-users.

##  Introducing inline execution containers

To address these challenges, we introduced 2 new methods for creating execution containers for deployments: 

- Using an inline Dockerfile
- Providing a URL to a Git repository

### Inline Dockerfile

With the inline Dockerfile option, you can define the container's build instructions in the step. 

![Execution Container from Inline Dockerfile](execution-container-from-inline-dockerfile.png "width=500")

This approach offers simplicity and immediacy. You can specify the dependencies and configuration settings without switching between different container images or repositories. 

It's particularly useful for smaller projects or scenarios where the Dockerfile is simple.

### URL to Git repository

This option lets you provide a URL to a Git repository containing the Dockerfile and associated resources. 

![Execution Container from Git URL](execution-container-from-git-url.png "width=500")

This approach offers greater flexibility and scalability. You can use existing Dockerfiles maintained in their version control system. 

By referencing a Git repository, you can: 

- Ensure consistency across projects
- Promote code reuse
- Take advantage of versioning and collaboration features provided by Git platforms

## Conclusion

Letting you choose between an inline Dockerfile or a URL to a Git repository simplifies deployment workflows with Docker. 

By offering flexibility and versatility, you can streamline the container creation process, integrate with your deployment process quicker, and foster collaboration through centralized code repositories. 

As organizations embrace containerization at scale, features like these accelerate development cycles and help you deliver value to your end-users faster.

Happy deployments!