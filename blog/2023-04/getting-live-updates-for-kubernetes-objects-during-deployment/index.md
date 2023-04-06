---
title: Getting Live Updates of Kubernetes Objects during Deployment
description: Introducing the Kubernetes object status check feature which provides a live update of Kubernetes objects during deployment.
author: yihao.wang@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - Product
  - Kubernetes
---

Kubernetes has become the de facto standard for container orchestration,
but deploying to a Kubernetes cluster is often a tricky process.
Once DevOps engineers have created Kubernetes objects in the cluster using manifest files,
they typically need to run a sequence of `kubectl get` or `kubect describe` commands,
until they confirm that the deployed objects are in fact up and running.

Good news is, you no longer have to do this everytime if you deploy to a Kubernetes cluster via Octopus.

The Kubernetes object status check feature will give you live updates during the deployment process 
with the statuses of all the Kubernetes objects you are deploying.
This provides you extra visibility to the cluster to help you verify Kubernetes objects statuses,
and detect any errors in the deployment as early as possible.

// insert a screenshot showing the overview of the new tab

Now, let's have a look at how this works.

## Using object status check for Kubernetes deployments

1. Register a cluster

    Before we start, we need a Kubernetes cluster as the deployment target. 
    If you are not familiar with the process,
    you can find instructions [here](https://octopus.com/docs/infrastructure/deployment-targets/kubernetes-target).

    In this post we are using a local cluster, 
    but this works for cloud clusters as well.

    // insert here a screenshot of Kubernete cluster configuration

2. Create a project

    Next, we will create a project that deploys to the Kubernetes cluster we just registered.

    Most built-in steps that deploys to Kubernetes clusters supports object status checks. 
    Exceptions are [Upgrade a Helm Chart](https://octopus.com/docs/deployments/kubernetes/helm-update) and "Run a kubectl CLI Script" (coming soon).

    In this demo, we will create a simple project that uses a [Deploy raw Kubernetes YAML](https://octopus.com/docs/deployments/kubernetes#raw-yaml-step) step.

    // insert here  a screenshot showing the Raw YAML step

    We going to create a Kubernetes deployment resource with 3 replicas that runs the nginx container.
    To do this, we put this YAML as inline script.

    // insert here a screenshot of inline source code editor with the YAML content

3. Configure the status check options

    You will notice that a new section has been added for the Kubernetes object status check.
    I will explain these options in more detail shortly.

    // insert here a screenshot of the new configuration

    For now, we just leave everything as default, 
    and that's pretty much everything you need to make it work!

4. View the live updates from the cluster

    Now, let's create a release and a deployment from this project.

    // insert here a screenshot that shows the button of create deployment

    Once the deployment starts, 
    you can find a new "Object Status" tab that has been added next to the "Task Logs" tab.
    Simply click into the tab and we will see the object status updates.

    // insert here a screenshot of the new tab header

    From the table, we can see one Kubernetes deployment resource, 
    one replica set, 
    and three pods.
    We did not define the replica set and the pods in the YAML file, 
    but since they are child resources to the deployment,
    Octopus shows them for you as well.

    // insert here a screenshot of the resource tables

5. Understand the settings

    Now we had a walk-through of the Kubernetes object status check feature, 
    let's revisit the configuration settings and understand how to configure them for your use case.

    // insert here a screenshot of the configuration section

    Whenever a new project is created,
    the Kubernetes object status check option is enabled by default.
    If you are re-configuring a project which has been created before the release of this feature,
    the Kubernetes object status check will not be enabled, 
    until you manually do so.

    You can also configure two optional timeouts.

    // insert here a screenshot of the step execution timeout section

    The "Step execution timeout" is the total time allowed for all Kubernetes objects in the action to be deployed.
    If there are any resources that are not in a successful state by the end of this timeout period,
    the step will stop executing and will be marked as failed.
    You can disable this timeout if you don't want to set a time limit.

    // insert here a screenshot of the stabilization timeout section

    The "Status stabilisation timeout" adds more stability to your deployment.
    Sometimes a Kubernetes object can have temporary failures but will self-heal eventually.
    For example, a pod may fail to spin up due to a temporary connection issue to the container registry,
    but it will be created successfully when the internet connection is back.
    You can use the stabilisation timeout to prevent this kind of temporary failures to cause a failed deployment.
    When this timeout is enabled, Octopus will wait for the period configured after the step fails or succeeds.
    The step will be marked as failed or succeeded only if the status does not change throughout this timeout period.


## Caveats

This feature can be super handy for Kubernetes deployments, but there are some caveats worth calling out.

First, the object status check tab only updates during the deployment process.
Once the deployment succeeds or fails, Octopus does not do further checks for the deployed resources.
This means any later updates to those objects, either done manually or by another deployment, won't be reflected in the table.

Second, if you are deploying with a "Deploy Kubernetes containers" step,
there is an existing option "Wait for the deployment to succeed" that allows you to wait until the completion of the deployment.
This option is not compatible with object status check feature because it uses the `kubectl rollout status` command under the hood.
We recommend not using this option for any new deployments and use the object status check feature instead.

Finally, we currently do not support object status checks on resources deployed via the [Upgrade a Helm chart step](https://octopus.com/docs/deployments/kubernetes/helm-update) step, 
or deployments that has been configured with a [blue/green strategy](https://octopus.com/docs/deployments/kubernetes/deploy-container#bluegreen-deployment-strategy). (But don't worry, we are planning to add the support for these shortly.)

## Conclusion

The Kubernetes object status check feature provides live updates during the deployment process for the Kubernetes objects being deployed.
It provides more visibility, gives you more confidence that your deployment is up and running,
and helps you detect and identify any errors in the deployment as early as possible.

We hope you like this feature. Please don't hesitate to share your experience or any recommendations with this feature [here](/)!

> The Kubernets object status check feature will be available for Octopus cloud users from early May. But if you want to try it now, please [contact us](/)!

Happy deployments!
