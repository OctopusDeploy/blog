---
title: Autoscaling Octopus workers using Kubernetes
description: Learn about our new worker that makes it easier to scale your deployment infrastructure. The worker runs on Kubernetes but helps with any deployment tasks to any target.
author: trent.mohay@octopus.com
visibility: public
published: 2024-08-26-1400
metaImage: img-blog-octopus-k8-worker-2024.png
bannerImage: img-blog-octopus-k8-worker-2024.png
bannerImageAlt: Octopus Deploy logo connected to a Kubernetes-branded box with an Octopus logo in it.
isFeatured: false
tags: 
  - Product
  - Kubernetes
  - Workers
---

We're excited to announce a new worker that makes it easier to scale your deployment infrastructure. While the worker runs on Kubernetes to provide the scalability you need, it runs deployment tasks for any deployment target.

This feature follows on from the recently released [Octopus Kubernetes agent](https://octopus.com/blog/kubernetes-agent). The Kubernetes agent was our "contact on the inside", simplifying deployments into a Kubernetes cluster. It allows deployments to execute Kubernetes commands from *within* the cluster. 

Now, we've built a Kubernetes worker – an extension to the agent that can execute *any* workload in the cluster, not just Kubernetes commands. 

In this post, I explain how it works and how you can use it. 

## Background

To appreciate how this new worker helps you, it's worth explaining the types of machines involved in a deployment: 

1. Deployment targets - machines that host runtime software packages
2. Workers - compute resources to execute the deployment process. 

We'll only deal with workers here – computers needed during a deployment that are otherwise idle. 

When using a physical or virtual machine as a worker, it must be adequately provisioned to handle peak-deployment loads. Unfortunately, when no operations are running, the worker machine holds onto those resources. 

With a Kubernetes worker, after a deployment is complete, the hardware resources get returned to the cluster. This lets the cluster repurpose or release them. 

The Kubernetes worker executes each deployment task in a new Kubernetes Pod (known as horizontal scaling). More resources are automatically provisioned when there's enough pressure on the cluster. Then, as work gets completed and the pods terminate, the additional resources get returned to the cluster. This reduces the running and maintenance costs associated with a fleet of physical (or virtual) worker machines.

## Is the new worker for you?

Until now, resources for workers could come from a variety of places: 

1. Directly from your Octopus Server instance. (This is a great place to start and acceptable for smaller installations, but it can be limiting in large systems.)
2. You may be using the dynamic workers supplied by the Octopus Cloud platform if you're cloud-hosted.
3. Self-managed VMs or physical machines. 

The Kubernetes worker may not reduce complexity for the first 2 groups. But, if you're in group 3, you may find the Kubernetes worker lets you replace a fleet of worker machines with a single Kubernetes worker that scales with your workloads. You don't need to deploy to Kubernetes or be an expert Kubernetes user to benefit from the Kubernetes worker.

## How to install the Kubernetes worker

You can install the Kubernetes worker using the Kubernetes agent [Helm chart](https://hub.docker.com/r/octopusdeploy/kubernetes-agent). You can do this manually via the command-line, or, far simpler, use the installation wizard we provide in the Octopus web portal.

This wizard guides you through the steps to capture values defining your worker, and provides the Helm command to install the worker.

1. In the sidebar menu, navigate to **Deploy**, then **Workers** under **Infrastructure**, then **Add Worker**, then **Add Worker**, and then **Kubernetes Worker**.

![Add Kubernetes worker](add-kubernetes-worker.png)

2. This launches the Kubernetes worker installation wizard. Here, you can name the worker and specify (and create) the worker pool(s) it belongs to. To create a new worker pool, go to the **Select worker pools** field and click the **+** symbol to dropdown the pool creation fields.

![Set Kubernetes Worker Properties](add-kubernetes-worker-properties.png)

3. The Kubernetes worker needs a shared filesystem. You can provide an existing storage class (under **Show Advanced**) or use a default Network File System (NFS) storage pod. The NFS storage pod is an easy, low-configuration storage option that doesn’t need extra volumes or storage classes configured in the cluster first. To use NFS storage, you must install an extra Helm chart.

![Install NFS Helm Chart](install-nfs-helm-chart.png)

4. At the end of the wizard, Octopus generates a Helm command that you copy and paste into a terminal connected to the target cluster. After it’s executed, Helm intsalls all the resources and starts the agent.

![Install Kubernetes worker Helm chart](install-kubernetes-worker-helm-chart.png)

5. If left open, the installation dialog waits for the worker to establish a connection and run a health check. Once successful, the Kubernetes worker is ready for use!

![Kubernetes worker installed successfully](kubernetes-helm-chart-installed-success.png)

## How does the worker communicate?

The Kubernetes worker uses the same polling communications protocol as Octopus Tentacle. It lets the worker connect from the cluster to Octopus Server (via proxy as needed), solving potential network access issues.

## In-built tooling 

When work-packages are sent to the Kubernetes worker, the actual operation runs in an Octopus Deploy [worker-tools](https://hub.docker.com/r/octopusdeploy/worker-tools) container.

If worker-tools is not appropriate for your workloads, you have 2 options:

1. Override the default container on the specific deployment step (as per standard workers).
2. Change the image used by all worker operations via the chart's values.

## How to customize the Kubernetes worker

The installation wizard creates a Kubernetes worker that is appropriate for the majority of workloads.

For the rest, manual customization is available.

You can configure many aspects of the worker via its values. For now, you need to perform these customizations via the command-line using a Helm upgrade command (or setting them manually during install).

For the full list of customizations, you can refer to [helm chart README.md](https://hub.docker.com/r/octopusdeploy/kubernetes-agent).

## How to try the Kubernetes worker

The Kubernetes worker works in a variety of clusters. If it's something you'd like to try, we recommend using a lightweight Kubernetes cluster, like:

* [Minikube](https://minikube.sigs.k8s.io/docs/start/?arch=%2Fmacos%2Farm64%2Fstable%2Fbinary+download)
* [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/)

As part of install, each of these configures a Kubernetes context in your terminal, allowing helm to connect to the installed cluster.

## When can I use it?

The Kubernetes worker is available for Cloud customers now as an early access preview. It will be available soon for our self-hosted customers in 2024.3.

## Learn more

The Octopus Deploy documentation covers the use of the Kubernetes worker.

To see the worker in action, register for our webinar on Tuesday, September 3, 2024, [Scalable deployment infrastructure with our new worker](https://streamyard.com/watch/ifsZdkjcrfCp).

Happy deployments!