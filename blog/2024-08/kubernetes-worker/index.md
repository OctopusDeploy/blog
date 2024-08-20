---
title: Autoscaling Octopus Workers - via Kubernetes
description: The Kubernetes agent is generally available.
author: trent.mohay@octopus.com
visibility: public
published: 2024-08-22-1400
metaImage: img-blog-octopus-k8-2024.png
bannerImage: img-blog-octopus-k8-2024.png
bannerImageAlt: Octopus Deploy logo connected to a Kubernetes-branded box with an Octopus logo in it.
isFeatured: false
tags: 
  - Product
  - Kubernetes
  - Workers
---
The words are literally made to go together - but it took a bit of work to make it all happen.

The story starts with some work we recently completed - the [Octopus Deploy Kubernetes Agent](https://octopus.com/blog/kubernetes-agent) - a lightweight installation agent which greatly simplifies how you deploy applications to a Kubernetes Cluster from OctopusDeploy.

Turns out - the Kubernetes Agent was a great platform to extend into a general workload executor - especially when considering how it would lean into Kubernetes ability to scale!

# Background
To appreciate the greatness of this change, it’s helpful to understand how Octopus Deploy executes deployments.

In a deployment, there can be two types of machine required:
1. Deployment Targets - machines which host runtime software packages
2. Workers - compute resources required to execute aspects of a deployment process

We’re just going to deal with Workers here - i.e. computers required to execute a deployment, but are otherwise idling.

This is where the Kubernetes Worker comes in - a worker which automatically scales up and down based on demanded workload. Thus reducing the running, and upkeep costs associated with physical and virtual Octopus workers.

# Kubernetes Autoscaling Worker
The Kubernetes Worker is a specialized kubernetes-specific tentacle (add link) - a lightweight application responsible for executing work-packages received from an OctopusServer.

On reception of a work-package, the Kuberentes Worker creates a new Kubernetes Pod using the requested container (or a sensible default), executes the work/script, and feeds the log output back to the requesting server.

Every work package gets a new pod - which eventually applies pressure to the cluster - causing new nodes to be added as required.

As work completes, and the pods terminate, resource requirements reduce, and the cluster can release the excess nodes.

# Is this for you?
OctopusDeploy provides a number of ways for you to provide Worker compute resources

1. Taking it directly from your literal Octopus Server instance
iGreat place to start, and fine for smaller installations, but can be limiting
1. If cloud-hosted, you may be using the Dynamic Workers supplied in cloud
1. Self-managed VMs or physical machines

The Kubernetes worker is unlikely to reduce complexity for the first two groups - but the third group? Users maintaining physical/virtual machines as workers - this will help. A lot.

The Kubernetes Worker will allow you to consolidate your virtual machines into a single worker, which can expand to fill the available cluster space.

# How To Install the Kubernetes Worker
The Kubernetes Worker can be installed via the Kubernetes Agent [helm chart](https://hub.docker.com/r/octopusdeploy/kubernetes-agent) - you can do this manually via the command-line, but a far simpler method is to use the “installation wizard” provided by the OctopusDeploy Web portal.

This wizard will guide you through a series of steps to capture values defining your worker, and provide the Helm command required to install the defined worker.

1. In the sidebar menu, navigate to Deploy → Infrastructure → Workers → Add Worker → Add Worker → Kubernetes
![Add Kubernetes Worker](add-kubernetes-worker.png)

2. This launches the Kubernetes Worker installation wizard. Here you can name the worker, specify (and create) the worker pool(s) to which it should belong. To create a new Worker Pool, select the “+” symbol to dropdown required pool creation fields.
![Set Kubernetes Worker Properties](add-kubernetes-worker-properties.png)

3. The Kubernetes worker needs a shared filesystem. You have the option of providing an existing storage class (under “Show Advanced”) or using a default Network File System (NFS) storage pod. The NFS storage pod is an easy, low-configuration storage option that doesn’t need extra volumes or storage classes to be configured in the cluster first. To use NFS storage, you must install an extra Helm chart.
![Install NFS Helm Chart](install-nfs-helm-chart.png)

4. At the end of the wizard, Octopus generates a Helm command that you copy and paste into a terminal connected to the target cluster. After it’s executed, Helm intsalls all the required resources and starts the agent.
![Install Kubernetes Worker Helm Chart](install-kubernetes-worker-helm-chart.png)

5. If left open, the installation dialog waits for the worker to establish a connection and run a health check. Once successful, the Kubernetes worker is ready for use!
![Kubernetes Worker Installed Successfully](kubernetes-helm-chart-installed-success.png)

# How does it communicate?
The Kubernetes Worker uses the same polling communications protocol as Octopus Tentacle.It lets the worker connect from the cluster to Octopus Server (via proxy as required), solving potential network access issues.

# Inbuilt Tooling 
When work-packages are sent to the Kuberentes Worker, the actual operation is executed within a Pod which is using Octopus Deploy’s [Worker-tools](https://hub.docker.com/r/octopusdeploy/worker-tools) image.

# How to Optimize the Kubernetes Worker

# Easy Ways to Try It Out!

# When do I get to use it?
It’s on Cloud instances already, and will be available to self-hosted customers in 2024.3 - so give it a few weeks.

# More Information & Support
The OctopusDeploy documentation suite covers the use of the Kubernetes Worker, otherwise reach out to our community slack, or direct to our support in case of trouble.

