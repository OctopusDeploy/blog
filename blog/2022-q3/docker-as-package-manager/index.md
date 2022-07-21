---
title: Using Docker as a package manager
description: This post demonstrates how Docker is a convenient solution for downloading and running many CLI tools
author: matthew.casperson@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: octopus-docker.png
bannerImage: octopus-docker.png
bannerImageAlt: The Octopus and Docker logos side by side.
isFeatured: false
tags: 
  - DevOps
---

A constant challenge when automating DevOps tasks is ensuring you have the correct tools for the job. Linux users have long enjoyed the ability to install tools from extensive and well maintained package repositories, while macOS users have HomeBrew and MacPorts, and Windows users have Chocolatey and winget. However, an increasing number of cloud based tools, such as platform specific CLI tools (eksctl, aws-iam-authenticator, kubectl, kind, helm, etc.) can only be reliably installed across multiple operating systems and Linux distributions via a direct binary download. Finding and downloading tools is a point of friction when automating DevOps tasks.

Docker provides a solution by allowing CLI based tools to be run from a Docker image. In this post, we'll explore how Docker can be used as a universal package manager to download and run many CLI tools across multiple operating systems.

## Find the right tools to provide DevOps solutions

DevOps is all about repeatability and automation, and frequently requires scripting tasks using specialized CLI tools. Many of the environments where scripts are run are ephemeral in nature. Examples include CI/CD agents that are created and destroyed as needed, Kubernetes pods used to run jobs, and virtual machines that scale up and down in response to demand. Often this means script writers can not assume CLI tools are installed, or depend on a particular version of a tool being available.

As we noted in the introduction, many of the tools DevOps teams require only provide direct binary downloads. This means even a simple script may need to first find the appropriate download link for any required CLI tools, download them (usually with some kind of retry logic to deal with network instability), extract or install the tools, and then finally run them.

This task is complicated by the fact that the tools required to download files from the internet differ between operating systems. Linux and macOS users may be able to count on tools like `curl` or `wget` being installed, while Windows users will likely use PowerShell CmdLets.

But what if there was a better way?

## Docker as a universal package manager

Every major CLI tool and platform provides a well maintained Docker image these days. Whether the images are published by vendors themselves on repositories like [Docker Hub](https://hub.docker.com/) or maintained by a third party like [Bitnami](https://bitnami.com/), there is a good chance you will find an up to date version of a CLI tool you need as a Docker image.

The nice thing about running Docker images is that the commands to download an image and execute it are the same across all operating systems and for all tools.

To download an image, reusing any previously downloaded images and with automatic retries, run the command:

```bash
docker pull image-name
```

The image is then executed with the command:

```bash
docker run image-name
```

As we'll see in the next sections, there are additional arguments required to reliably run images for short lived management tasks, but generally speaking these two commands are all you need to know for every tool and operating system. This is huge improvement over writing unique scripts between Windows and Linux to download binary files from custom URLs.

## A practical example

Let's look at helm as a practical example of running CLI tools from Docker images. The first step is to download the image locally. Note this step is optional, because Docker will download missing images before running them:

```bash
docker pull alpine/helm
```

Run helm with no arguments with the following command. The `--rm` argument passed to `docker` cleans up the container when it is finished, which is desirable when running images for single operations:

```bash
docker run --rm alpine/helm
```

This results in the help text being printed to the console, just as if you had run a locally installed version of `helm` with no arguments.

A common requirement of CLI tools used for DevOps automation is the ability to read files and directories, whether they are configuration files, larger packages like ZIP files, or directories with application code.

By their nature, Docker containers are self contained and by default do not read files on the host. However, local files and directories can be mounted inside a Docker container with the `-v` argument, allowing the processes run in a Docker container to read and write files on the host.

The command below mounts several shared directories with container cerated to run the `alpine/helm` image. This allows configuration settings, like helm repositories, to be accessed by the `helm` executable in the Docker container. It also passes the arguments `repo list`, which lists the configured repositories:

```bash
docker run --rm -v "$(pwd):/apps" -w /apps \
    -v ~/.kube:/root/.kube -v ~/.helm:/root/.helm -v ~/.config/helm:/root/.config/helm \
    -v ~/.cache/helm:/root/.cache/helm \
    alpine/helm repo list
```

If you have not previously defined any helm repositories, the output of this command will be:

```bash
Error: no repositories to show
```

To demonstrate how the volume mounting works we'll configure a new helm repository using a locally installed version of `helm`:

```bash
helm repo add nginx-stable https://helm.nginx.com/stable
```

Running the helm Docker image to list the repositories shows that it has loaded the repository added by the locally installed copy of `helm`:

```bash
NAME            URL
nginx-stable    https://helm.nginx.com/stable
```

This also works in reverse. Run the following command to add a second repository from the helm Docker image:

```bash
docker run --rm -v "$(pwd):/apps" -w /apps \
    -v ~/.kube:/root/.kube -v ~/.helm:/root/.helm -v ~/.config/helm:/root/.config/helm \
    -v ~/.cache/helm:/root/.cache/helm \
    alpine/helm repo add kong https://charts.konghq.com
```

Then list the repos from the locally with the command:

```bash
helm repo list
```

The local installation reflects the newly added repo:

```bash
NAME            URL
nginx-stable    https://helm.nginx.com/stable
kong            https://charts.konghq.com
```

## Aliasing docker run commands

While Docker provides a convenient ability to download and run images, the `docker run` command can become quite long tedious to type. Fortunately, it is possible to alias the `docker run` command so it can be used as a drop in replacement for a locally installed tool.

Use the `alias` command to map `docker run` to the `helm` command:

```bash
alias helm='docker run --rm -v $(pwd):/apps -w /apps -v ~/.kube:/root/.kube -v ~/.helm:/root/.helm -v ~/.config/helm:/root/.config/helm -v ~/.cache/helm:/root/.cache/helm alpine/helm'
```

The previous alias command only works in the session it was defined in. If you log out and back in, the alias is lost. To make the alias permanent, edit the `~/.bash_aliases` file, and add the same alias command above on a new line:

```bash
vim ~/.bash_aliases
```

In many distributions, the `~/.bash_aliases` file is loaded automatically by the `~/.bashrc` file. If not, add the following code to the `~/.bashrc` file:

```bash
if [ -f ~/.bash_aliases ]; then
    . ~/.bash_aliases
fi
```

## Using Docker images in Octopus script steps

If you are using aliases in a non-interactive shell (like running a script step in an Octopus deployment), you need to ensure aliases are expanded by running the following command:

```bash
shopt -s expand_aliases
```

The following snippet shows a script step downloading a Docker image and setting the aliases. The logging level has been set to verbose so the Docker image download messages don't fill the deployment logs. It also demonstrates passing environment variables to an aliased command via the `-e` arguments:

```bash
echo "Downloading Docker images"

echo "##octopus[stdout-verbose]"

docker pull amazon/aws-cli 2>&1

# Alias the docker run commands
shopt -s expand_aliases
alias aws='docker run --rm -i -v $(pwd):/build -e AWS_DEFAULT_REGION=$AWS_DEFAULT_REGION -e AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID -e AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY amazon/aws-cli'

echo "##octopus[stdout-default]"
```

## Conclusion

Docker images provide a consistent and convenient way to install and run common DevOps CLI tools, especially when compared to the process of writing OS specific commands to download binary files from unique URLs. In this post we looked at how to run a Docker image from the command line or in a script, and provides some tips to allow Docker images to run as drop in replacements for locally installed tools.

## Learn more

If you are looking to build and deploy containerized applications to AWS platforms such as EKS and ECS, the [Octopus Workflow Builder](https://octopusworkflowbuilder.octopus.com/#/) populates a GitHub repository with a sample application built with GitHub Actions workflows and configures an Hosted Octopus instance with sample deployment projects demonstrating best practices such as vulnerability scanning and Infrastructure as Code (IaC). 

Happy deployments! 

