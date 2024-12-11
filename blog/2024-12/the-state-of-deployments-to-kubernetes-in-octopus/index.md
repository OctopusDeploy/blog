---
title: The state of deployments to Kubernetes in Octopus
description: An overview of the evolution of Octopus's capabilities for deployments to Kubernetes and a sneak peek into the future
author: nikita.dergilev@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - DevOps
  - Company
  - Product
  - Kubernetes
---

For the past few years, Kubernetes has been Octopus's main focus, and we plan to continue prioritizing it in 2025. We want to deliver the best experience for enterprises automating deployments to Kubernetes, especially at the scale of hundreds of microservices and many teams.

This post covers the challenges that enterprises encounter when deploying to Kubernetes at scale, and the current state of deployment best practices and tools. 

We also talk about Octopus's approach to Kubernetes deployments and recap the  capabilities we introduced in 2023 and 2024. We also walk through a scenario that shows how you can use these capabilities to improve the experience of deploying to Kubernetes at scale. Finally, you get a sneak peek into our plans for Kubernetes in 2025.

## Deployments to Kubernetes, challenges at scale

Kubernetes has influenced the way we develop and run applications. With Kubernetes’ capabilities, we can: 

- Easily break up large applications and run hundreds of microservices
- Create a bunch of environments dynamically
- Seamlessly scale resources
- Run applications in clouds, on edge devices, or in a private data center using one platform and one configuration

Not only that, but deployments have also evolved. Previously, a deployment would typically include steps like file shuffling, configuration changes, and service restarts. With Kubernetes, one human-readable manifest has all the information about the application and infrastructure configuration.

It's no surprise that Kubernetes is a new default platform for many organizations around the globe.

However, these new great capabilities also lead to a more intricate deployment landscape. The north star for any pipeline is still fully automated deployments from development to production. Automation lets you ship software multiple times a day you can deliver small and safe regular improvements. This lowers the cost of deployments due to the absence of manual testing and approvals.

Deploying monolithic applications was already a challenge, but now we're faced with the need to automate deployments for hundreds of microservices. These microservices run on the same cluster, which means they can affect one another. We also need to find a way to dynamically provision environments with the correct versions of these microservices and manage manifests that combine both application and infrastructure configuration.

Plus, existing deployment challenges remain. We need to run different sets of tests across various environments, comply with regulatory requirements, integrate with change management tools, and more. But now, we have to address these issues for a significantly larger number of applications.

The challenge for modern DevOps and Platform teams is to find the right set of tools to automate deployments. On the one hand, these tools should be capable of managing deployments at scale. On the other hand, they should be easy to adopt at the very beginning when you’re starting with just a couple of applications and a few teams on Kubernetes. You really need to plan for the future and consider questions like:

- Where will you deploy (the number of environments, regions, clusters)?
- Who will own deployments (and how easy will it be to self-serve)?
- How many applications will be deployed?
- What extra requirements does your organization have?

If you want to learn more, I recorded [a webinar about deployments to Kubernetes at scale](https://youtu.be/uXUG8s4sFMY?si=PTGchl4U1uTctHYO).

## CD problems and tools evolution

GitOps has been the main topic in Kubernetes deployments for a few years now. Argo CD is the leading OSS project in this space. Flux CD, the second mature project, is still popular, but it has noticeably lower adoption. Other projects like Jenkins X or Fleet are niche products.

This year, the focus shifted from the GitOps implementation in general to a more narrow problem of environment promotions with GitOps.

As organizations scaled on Kubernetes, they realized that OSS tools don’t have a good solution for promotion orchestration. Until recently, the only way to automate the environment promotion with GitOps tools was to build in-house automation on tools like Jenkins or GitHub Actions. However these solutions don’t scale well and are expensive to maintain long-term.

This year, the market has reacted to the challenges of environment management with GitOps OSS by introducing several specialized tools. Codefresh GitOps was the first of these tools and is the most developed project. Meanwhile, Kargo and Argo CD GitOps Promoter aim to address the same issues, but are still in their early stages.

In addition to the trend of developing GitOps OSS tools to tackle environment promotion challenges, there's another movement where sophisticated CI/CD tools are adopting more GitOps principles. GitOps significantly enhances the developer experience. It uses the tools we already use to manage code and provides greater confidence in achieving the desired state. Building on this value is a logical progression. In the following sections, we touch on how you can implement the code-first approach in Octopus.

## Octopus and Kubernetes

Octopus Deploy was designed to simplify complex deployments across various environments. This is Octopus's core job, and we do it well.

Since 2018, Octopus has focused on developing in-built Kubernetes capabilities, and it's been our top priority for the past 2 years. We're simplifying Continuous Delivery for Kubernetes. By "simple", we mean a low cost of ownership, straightforward configuration, easy maintenance, and peace of mind for everyone involved.

We believe organizations should use one tool and set of practices for all their deployments, instead of starting from scratch and building a completely new stack for each hosting platform.
We simultaneously adopt Kubernetes-native tools, like Helm and Kustomize, along with best practices like declarative configurations and using Git as the source of truth. Our goal is to integrate these tools and practices into Octopus, making it easier to incorporate them into deployment pipelines.

Kubernetes introduced a few new challenges. One of the biggest challenges is that a deployment doesn't finish when you apply a configuration to a cluster. Instead, the cluster should achieve the desired state, eventually. At the same time, knowing the deployment status is vital. That’s why our focus right now is to bring more capabilities to verify deployments to Kubernetes in Octopus. See the sections below to learn more.

Currently, over 400 organizations use Octopus for regular deployments to Kubernetes. More than 300 of these deploy every week, and over 250 deploy daily.

These organizations manage more than 8,000 projects and run approximately 600,000 Kubernetes steps each month.

24% of these projects get deployed with our built-in Helm step and 18% with the plain YAML step. For 18% of the projects, our customers decided not to manage manifests outside of Octopus and configured them instead with our **Configure and apply Kubernetes resources** step.

## New Kubernetes features in Octopus during 2023 and 2024

Let’s revisit 2023 and 2024 to see some of the ways the experience with Kubernetes has changed in Octopus.

__[Kubernetes Object Status](https://roadmap.octopus.com/c/73-kubernetes-object-status-check)__

With the release of 2023.2, Kubernetes Object Status became one of the most popular features in Octopus. Now, deployments in Octopus no longer end with a simple manifest application. Instead, Octopus can interpret YAML manifests and monitor the status of objects until the cluster achieves the desired state.

With this feature, the success or failure of a deployment in Octopus also reflects if the desired configuration has successfully started on the target. Octopus also provides a list of deployed objects along with basic information and health status at the time of deployment. This removes the need to check other tools for deployment verification.

__[Kubernetes Object Status for Helm](https://roadmap.octopus.com/c/171-kubernetes-object-status-for-helm)__

Recently, we enhanced the Kubernetes Object Status by adding support for Helm. Since Helm already includes built-in capabilities to verify deployment success (using the `--wait` argument), we didn’t want to replace this functionality. Instead, we improved its discoverability and enabled it by default for the newly added steps. Octopus can also display the object list, health, and details for Helm in the same way it does for all Kubernetes steps.

__[Sourcing Kubernetes Configuration Files from Git](https://roadmap.octopus.com/c/43-sourcing-kubernetes-configuration-files-from-git)__

Kubernetes manifests are essentially code, and the most effective way to manage code is through Git. So, we streamlined the process between Git and Octopus. Octopus now directly sources files from Git. This feature is available for all built-in Kubernetes steps and for script steps.

__[Smart selection for Helm values sources](https://roadmap.octopus.com/c/114-smart-selection-for-helm-values-sources)__

Later, we incorporated sourcing from Git for Helm charts and values files. We also reimagined the entire process of managing values files in Octopus. Now, you can link as many files as you need and specify the order they should apply. This facilitates at-scale scenarios, as demonstrated in the example below.

__[Kustomize step](https://roadmap.octopus.com/c/64-kustomize-step)__

Kustomize is widely used for scaling Kubernetes configurations. Many customers prefer not to use Helm and told us they wanted a more advanced templating engine in Octopus. We knew that combining Kustomize with Octopus variables would enable users to apply a single configuration template across hundreds of projects. This prompted us to add the built-in Kustomize step in 2023.3.

__External feed triggers for [Helm](https://roadmap.octopus.com/c/72-trigger-release-creation-on-helm-chart-updates) and [container images](https://roadmap.octopus.com/c/69-trigger-release-creation-on-container-image-update)__

In 2024.2, we introduced external feed triggers. These new triggers let Octopus automatically create releases whenever a new version of a container image or Helm chart gets published. This is a crucial step towards implementing GitOps principles in Octopus and supporting at-scale scenarios. The triggers also simplify CI pipelines and make it easier to track changes in third-party applications.

__[Git triggers](https://roadmap.octopus.com/c/68-git-triggers-for-release-creation)__

We recently released another trigger. Now, Octopus can monitor Git repositories for changes and create releases automatically upon updates. This enables advanced scenarios, like simultaneous updates for multiple projects if a configuration template is modified, or creating a release when Helm values files change.

__[Kubernetes agent](https://roadmap.octopus.com/c/84-octopus-agent-for-kubernetes)__

2024.2 was a big release for Kubernetes in Octopus. Along with the new triggers, we launched the Kubernetes agent. This is our largest enhancement to the Kubernetes experience in Octopus to date (but this won't be the case for long). The agent allows for workerless deployments to Kubernetes, greatly enhances cluster authentication, improves security, and simplifies network configuration. We recommend using the agent as the default method for connecting Octopus with Kubernetes clusters.

__[Kubernetes worker](https://roadmap.octopus.com/c/108-kubernetes-worker-new-autoscaling-worker-on-kubernetes)__

The Kubernetes agent allows for efficient use of cluster resources by scaling up and down based on deployment load changes. We received extensive feedback requesting the same flexibility for all deployments. As a response, we released a new Kubernetes worker. This scalable worker operates on Kubernetes and can deploy to any target or execute any runbook. Additionally, it provides a solution for customers who cannot install Kubernetes agents on their clusters, such as those with small edge clusters where every megabit of RAM is critical.

These aren't even all the Kubernetes improvements released in the last 2 years. See our [roadmap for more](https://roadmap.octopus.com/tabs/3-released).

## How to automate deployments to Kubernetes at scale with Octopus

Let’s use an example to put all the Octopus capabilities into a scenario of deployment to Kubernetes at scale. This is just one of the ways to configure Octopus, of course, there are so much more options available.

Let’s paint a picture together. Imagine we are part of a company with dozens of developer teams — a significant player in the SaaS market, known for delivering top-notch products (we won’t specify what market it is). We are harnessing the full power of Kubernetes, deploying across multiple environments, and managing production clusters in various regions. Our organization handles over 500 microservices, with many of them being deployed multiple times a day. While our engineers are exceptional, they aren’t necessarily Kubernetes experts; however, thankfully, our Platform team is among the best in the industry.

Compliance and security are paramount in our environment. Our clients trust us, and we take that responsibility seriously. 

So, how can Octopus help us streamline our deployments while ensuring they remain secure? Let’s explore this!

![A project per microservice linked to one Helm chart with invividual per-project values files and Octopus variables for environment-specific parameters](./Helm-at-scale-diagram.jpg "width=500")

### Configuration templating with Helm

How can we simplify the process of adding a new microservice to our deployment landscape? The first step is to provide a centralized configuration template — a single manifest that can be used across all our microservices while still allowing for modifications as needed. Our goal is to strike a balance between simplicity, flexibility, and control.
Helm is an ideal solution for this. The Platform team can create a Helm chart that encompasses all the 
configuration scenarios required by the Application teams. By using values files, the Application teams can easily enable or disable certain configuration elements, such as enabling Ingress or adding PersistentVolumes.

For more complex scenarios, we can create a new Helm chart that includes the original chart as a dependency. 

Additionally, it’s beneficial to store the Helm chart in an OCI registry to take advantage of versioning. The Helm values files can be stored in the application repositories alongside the application code. This setup also clarifies ownership, as the Platform team can manage the Helm chart independently from the values files that are owned by the Application team.

Creating a new pipeline becomes straightforward: we simply add the Helm step, point it to the Helm chart, and copy a values file template to our application repository.

### Releasing new version

With this configuration, we have several options for releasing new versions of our applications. We can either update the values files each time we release a new container image or utilize Octopus’s built-in mechanism to source container information as a variable.

There’s no definitive right or wrong approach. Using Octopus variables means you won’t have to frequently update configuration files. However, storing container information in the values files allows you to maintain a history of changes in Git.

### Simple environment configuration

Our environments are similar, but there are important differences between them. We need to implement slight changes, which may not be specific to an application. For instance, these adjustments could involve an environment label or a database URL.

Should we create a separate values file for each environment? Probably not. With Octopus, we can incorporate Octopus variables directly into the values file or Helm charts.

We can also create a variable set, which is a group of variables shared among multiple projects, to manage non-specific parameters. Additionally, we can define a few project-specific variables to handle application-unique configurations, such as the number of replicas.

Furthermore, we can utilize Octopus system variables to automatically include information like releases, environments, and project names in our manifests when needed.

### Kubernetes Object Status for deployment verification

By enabling the `--wait` option for Helm deployments, our developers can rely on the deployment status in Octopus to ensure that everything went as planned. In many cases, we prefer not to grant direct access to production clusters or cloud interfaces, making Octopus the simplest way to verify deployments.

Additionally, we cannot expect every developer to be familiar with Helm charts. Therefore, by utilizing the Kubernetes Object Status, they can easily inspect all deployed objects and their key properties to confirm their deployment configuration.

### Triggers to automate deployments

With 500 microservices, deployment orchestration can be challenging. We have two key tasks to address: first, we need to independently deploy each microservice when a new version is ready. Second, there may be instances where we need to redeploy all the microservices, such as when we introduce a critical change to our Helm chart.

To automate this process, we can create an external feed trigger for each microservice project, pointing to its corresponding container image. Additionally, we can establish an orchestration project that manages releases and initiates deployments for all or selected microservice projects. This orchestration project can include a trigger that points to the Helm chart.

You can see how it works in practice in our [Samples instance](https://samples.octopus.app/app#/Spaces-105/projects?page=1&pageSize=50). The orchestration project is the first project in each project group (Helm/Azure, Kustomize/AWS, or YAML/GCP). All other projects within the group consist of individual microservices. You can find the configuration for container image triggers in the `Project` > `Triggers` section.

### Adding new targets

Finally, what if our 500 microservices run across multiple clusters? For instance, we might have a dedicated cluster for each customer or branch. In this scenario, we can expect to frequently provision new clusters and want each new cluster to have the latest versions of the microservices immediately upon creation.

To streamline this process, the first step is to add a Kubernetes agent during the cluster provisioning. Once the agent is installed, it will automatically register the cluster with Octopus.

You can also configure deployment target triggers to initiate deployments to a target as soon as it becomes available. This eliminates the need for manual intervention after adding a new cluster. With Octopus, it’s possible to install all necessary system applications (cluster bootstrapping) as well as any software you want to run on the cluster.

## What’s coming in 2025?

2025 is poised to be an exciting year for Kubernetes in Octopus. Currently, we are focusing on a major enhancement: [Kubernetes Live Status](https://roadmap.octopus.com/c/122-kubernetes-live-object-status). With this feature, Octopus will continuously monitor deployed objects even after deployment, providing real-time insights into the health of your applications directly on the Octopus dashboard. Additionally, Octopus will notify you if the cluster configuration drifts from what was deployed with a release. You will also have access to details about objects, events, and logs.

We are also developing [manifest inspection](https://roadmap.octopus.com/c/144-kubernetes-manifest-inspection). This feature will complement the live status by allowing users to view and compare YAML manifests for all deployed objects.

Together, these improvements will equip you with a robust toolset for deployment verification and troubleshooting in Octopus.

But wait, there’s even more on the horizon! We’re diving headfirst into the exciting world of native integration with [Argo CD](https://roadmap.octopus.com/c/85-argo-cd-octopus-agent-for-k8s-resource-monitoring-and-deployments) and [Argo Rollouts](https://roadmap.octopus.com/c/146-argo-rollouts-in-octopus) right here in Octopus land!

We can’t wait to sprinkle some Octopus magic on Argo CD deployments, bringing our awesome core features like release management, environment modeling, and lifecycle management into the mix. Adding Octopus to the existing Argo CD landscape should be easy and natural as Octopus will understand and leverage features like ApplicationSets and will use Argo CD observability.

We’re bubbling with excitement about the journey ahead this coming year, and we want you to be a part of it! Your thoughts are like little nuggets of gold for us, and we can’t wait to hear what you think. Don’t be shy — [drop your ideas on our roadmap](https://roadmap.octopus.com/tabs/1-under-consideration) and don’t forget to mention if you’re up for a friendly chat. Let’s make some magic together!

## Conclusion

Kubernetes has fundamentally transformed how applications are developed and managed, enabling organizations to efficiently scale and automate their deployments. At the same time, the complexity of managing hundreds of microservices and ensuring smooth deployments across various environments presents significant hurdles that must be addressed.

Throughout this post, we explored the evolution of Octopus’s capabilities for Kubernetes, highlighted Octopus's approach to facilitating these deployments, and reviewed a real-life example of how you can use Octopus to streamline deployments for hundreds of microservices with centralized configuration templating and flexibility at the same time.

Looking ahead, 2025 will be the most exciting year for Kubernetes in Octopus. We are thrilled about the upcoming features that will enable new use cases for Octopus.

Share your story about deployments to Kubernetes! What challenges have you faced? What tools or practices have worked for you?

Your feedback is invaluable as we continue to refine our offerings and support the community in this rapidly changing environment. Thank you for joining us on this journey!

## Learn more

- [Octopus roadmap](https://www.example.com/resource)
- [Webinar: Deployments to Kubernetes at scale](https://www.youtube.com/watch?v=uXUG8s4sFMY)
- [Kubernetes in Octopus documentation]()

## Watch the webinar: {5 ways to level up your Kubernetes deployments with Octopus}

<iframe width="560" height="315" src="https://www.youtube.com/watch?v=Eo03H_1VxGc&t=18s" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

We host webinars regularly. See the [webinars page](https://octopus.com/events) for details about upcoming events, and live stream recordings.

Happy deployments!
