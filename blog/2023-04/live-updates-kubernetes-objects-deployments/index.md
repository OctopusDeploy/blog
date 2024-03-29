---
title: Live updates of Kubernetes objects during deployment
description: Introducing the Kubernetes Object Status check feature, providing a live update of Kubernetes objects during deployment.
author: yihao.wang@octopus.com
visibility: public
published: 2023-04-26-1400
metaImage: blogimage-k8objectsstatusfeature-2023.png
bannerImage: blogimage-k8objectsstatusfeature-2023.png
bannerImageAlt: Octopus logo connected to two Kubernetes logo in front of a stylized Octopus dashboard
isFeatured: false
tags: 
  - Product
  - Kubernetes
---

Kubernetes has become the standard for container orchestration. [Deploying to a Kubernetes cluster](https://octopus.com/use-case/kubernetes) can be tricky, though. After you create Kubernetes objects in your cluster using manifest files, you typically need to run a sequence of `kubectl get` or `kubect describe` commands. You need to do this until you confirm the deployed objects are running.

We're pleased to have addressed this with our new feature, Kubernetes Object Status.

This feature gives you live status updates during the deployment process for all the Kubernetes objects you're deploying.

You get greater visibility of your cluster to verify the status of your Kubernetes objects and detect deployment errors as early as possible.

![Octopus release dashboard showing the Kubernetes object status check page](status-check-page.png)

In this post, I show you how our new Kubernetes Object Status feature works.

## Using the Kubernetes Object Status for deployments

### Step 1: Registering a cluster

Before you start, you need to register a Kubernetes cluster as the deployment target. 

You can follow the [steps in our docs](https://octopus.com/docs/infrastructure/deployment-targets/kubernetes-target) to do this.

In this post, I use a local cluster that runs on Minikube, but this process works for cloud clusters as well.

![Octopus UI showing a configured Kubernetes cluster as a deployment target](cluster-configuration.png)

### Step 2: Creating a project

Next, you need to create a project that deploys to the Kubernetes cluster you just registered.

Most built-in steps that deploy to Kubernetes clusters support object status checks. Exceptions are: 

- **[Upgrade a Helm chart](https://octopus.com/docs/deployments/kubernetes/helm-update)** (support for this step is coming soon) 
- **Run a kubectl CLI Script**

In this post, you create a simple project that uses a **[Deploy raw Kubernetes YAML](https://octopus.com/docs/deployments/kubernetes#raw-yaml-step)** step.

![Octopus UI showing the Deploy raw Kubernetes YAML step](raw-yaml-step.png)

You need to create a Kubernetes deployment resource with 3 replicas that run the NGINX container. To do this, use this YAML as an inline script:

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      labels:
        app: nginx
      name: nginx
      namespace: octopus
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
          - image: nginx
            name: nginx
    ```
    
:::hint 
For this post, I created the objects in the `octopus` namespace. Please remember to change this to the namespace that your deployment target is configured with.
:::

### Step 3: Configuring the status check options

There's a new section added for the Kubernetes Object Status check. I explain these options in more detail shortly.

:::hint 
We do not support object status checks for deployments configured using a [blue/green strategy](https://octopus.com/docs/deployments/kubernetes/deploy-container#bluegreen-deployment-strategy) yet. For these deployments, you can't select this option. We plan to add support soon. 
:::

![Octopus process editor showing the Kubernetes object status check configuration section](kubernetes-object-status-check-configuration.png)

For now, you can leave all the default options and it will work.

### Step 4: Viewing live updates from the cluster

Next, create a release and a deployment for this project.

![Octopus UI showing the page to create the deployment](create-deployment.png)

After the deployment starts, you find a new **KUBERNETES OBJECT STATUS** tab next to the **TASK LOG** tab.

Click **KUBERNETES OBJECT STATUS** to see the object status updates.

![Octopus UI highlighting the Kubernetes object status check header](object-status-tab.png)

From the table, you see one Kubernetes deployment resource, one replica, and 3 pods.

As you didn't define the ReplicaSet and the Pods in the YAML file, but they're child objects of the deployment, Octopus shows them for you as well.

![Octopus UI showing the deployed resources](resources-table.png)

### Step 5: Understanding the settings

Now you've seen how the Kubernetes Object Status feature works, I revisit the configuration settings and explain how to configure them for your use case.

Whenever a new project is created, the Kubernetes Object Status option is enabled by default, **Check that Kubernetes objects are running successfully**.

If you're re-configuring a project you created before the release of this feature, the Kubernetes Object Status check won't be enabled until you enable it manually.

![Kubernetes object status check options](kubernetes-object-status-check-options.png)

You can also configure 2 options:

- **Step timeout**

This timeout is the total time allowed for all Kubernetes objects in the action to deploy. If any resources defined in this step are not in a successful state by the end of this timeout period, the step stops executing and gets marked as failed. You can disable this timeout if you don't want to set a time limit.

- **Wait for Jobs to complete during deployment**

You find this option if the step you're deploying supports Kubernetes Jobs. This includes the **[Deploy raw Kubernetes YAML](https://octopus.com/docs/deployments/kubernetes#raw-yaml-step)** step, and the **Deploy Kubernetes containers** step with `Jobs` as the resource type.

When you check this option, Octopus waits for Kubernetes Jobs created in the deployment to complete or fail before finishing this step. Otherwise, Octopus treats Jobs as successful as soon as they're created without waiting for their execution.

By default, this option is unchecked. This is compatible with the current behavior, which doesn't wait for Jobs to execute.

## Caveats

This feature is helpful for Kubernetes deployments, but there are some caveats worth calling out.

1. The **KUBERNETES OBJECT STATUS** tab only updates during the deployment process.
After the deployment succeeds or fails, Octopus does not do further checks for the deployed resources. Any later updates to those objects, performed manually or by another deployment, won't be reflected in the table.

1. If you're deploying with a **Deploy Kubernetes containers** step, the option **Wait for the deployment to succeed** lets you wait until the deployment is complete. This option is not compatible with the Kubernetes Object Status feature because it uses the `kubectl rollout status` command. We don't recommend this option for new deployments. We recommend you use the Kubernetes Object Status feature for new deployments instead.

1. We do not support object status checks on resources deployed via the **[Upgrade a Helm chart](https://octopus.com/docs/deployments/kubernetes/helm-update)** step. We also do not support checks on deployments configured with a [blue/green strategy](https://octopus.com/docs/deployments/kubernetes/deploy-container#bluegreen-deployment-strategy). However, we plan to add support for these soon.

## Conclusion

The Kubernetes Object Status feature provides live updates during the deployment process for the Kubernetes objects being deployed.

It provides you with greater visibility, gives you more confidence that your deployment is running, and helps you detect and identify errors in the deployment as early as possible.

We'd love feedback on this feature while we continue to refine it. If you're an Octopus Cloud customer, you can register for the early access preview (EAP) available now. [Register for the Kubernetes Object Status EAP](https://oc.to/EAP).

Happy deployments!
