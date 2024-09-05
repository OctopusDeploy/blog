---
title: Prioritize Deployments
description: Prioritize your deployments. Fast-tracking deployments has never been easier. 
author: susan.pan@octopus.com
visibility: public
published: 2024-09-9-0900
metaImage: -750x400px.png
bannerImage: -750x400px.png
bannerImageAlt: 
isFeatured: false
tags: 
  - Product
  - Automation
---

As our customers grow, Octopus endeavors to ensure our tools allow teams to deploy what they need, when they need it. We're excited to introduce two methods to prioritize essential deployments. This feature expands on the 2023.4 feature which provides the ability to manually prioritize a deployment. Now, we've automated the prioritization process, ensuring essential deployments are expedited.

In this post, I explain how to configure priority deployments.


## Prioritize deployments

We've introduced two ways to automate deployment priority.

1. Prioritize the deployment when creating a new deployment

2. Prioritize an environment in a lifecycle phase. All deployments to that particular environment(s) in the lifecycle will be prioritized.


### Prioritize when creating a new deployment

When creating a new deployment, configure its priority before deploying. By checking the **Priority** checkbox and deploying, your deployment will be fast-tracked to the top of the task queue.

![Configure priority when creating a new deployment](priority-deployments-create-deployment.png)


### Prioritize an environment in a lifecycle phase

This option is beneficial if you want to prioritize all deployments to a specific environment. When configuring your lifecycle, check the Priority checkbox under Phase Priority. This will ensure all deployments deployed to that lifecycle phase will be prioritized in the task queue.

![Configure priority for an environment in a lifecycle phase](priority-deployments-lifecycle-phase.png)


### Manually Prioritize a deployment task

This was introduced in Octopus 2023.4 and serves as an excellent option to fast-track your deployments when they are already waiting in the task queue. On the tasks page, select the overflow menu for the task you want to prioritize, then select move to top. Your task will move to the top of the queue.

![Fast-track your deployment by manually moving to top](priority-deployments-move-to-top.png)


## Conclusion

Priority deployments are available for Cloud customers as an early access feature.


## Learn more

Read about [Prioritizing Tasks in our docs](https://octopus.com/docs/tasks/prioritize-tasks).

Happy deployments!



