---
title: Re-prioritize Task Queue
description: A blog post detailing a new feature that allows Octopus users to re-prioritize the task queue. This new feature will be available from Octopus version 2023.4 and onwards.
author: ian.khor@octopus.com
visibility: private
published: 
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - product
  - tasks

---

The Octopus task queue operates as a first in, first out (FIFO) system. Additionally, there are many different types of tasks in Octopus, such as deployments, runbacks, system tasks etc. The FIFO task system operates on an overall instance level and requires each task to be completed before the next can begin. The task queue, in combination with the task cap, ensures that users are able to not only view all the tasks that are running through the instance, but to also understand the limit on the number of tasks due to the task cap.

A huge pain point about the FIFO nature of the task queue is the user's inability to re-prioritize certain tasks to take precedence over others that are not as important. Examples of these sorts of tasks include hot fixes, production-level deployments or runbooks that need to run first before anything else can run in the queue.

We addressed this by introducing a new feature that allows users to re-prioritize tasks in both the Task Overview dashboard, as well as the Deployment page for a specific release in a project

In this post, I will show you how to re-prioritize your task queue from the space level Task Overview dashboard and the Task page for a particular release.

## The ability to move a queued task to the top from the Task Overview dashboard
[GIF here]

The ability to re-prioritize the task queue can be found on the space-level Task Overview dashboard. Customers are able to move a queued task to run next by:

- Identifying the queued task they would like to run next;
- Selecting the burger menu on the right hand side of the selected task; and
- Clicking on ‘Move to Top’

The queued task will then move to the top of the queue and will run next after the task that is currently being executed. This will not do anything to the task that is currently executing and only changes the order of which queued task executes next after the task that is currently being executed.


## The ability to move a queued task to the top from the Task page for a particular release
[GIF here]

Additionally, the ability to re-prioritize the task queue can also be found on the Task page for a particular release. Customers are able to move a queued task to run next by:

- Identifying the tasks in a particular release for a project that you want to run next;
- Go to Projects on the top navigation bar;
- Go to Releases on the left hand side navigation bar; and
- Click on ‘Move to Top’ on the top right hand corner of the screen

The queued task will then move to the top of the queue and will run next after the task that is currently being executed. This will not do anything to the task that is currently executing and only changes the order of which queued task executes next after the task that is currently being executed.

## Conclusion

The new ability to re-prioritize the task queue allows you to move critical and important queued tasks to the run next in a simple, efficient and inquitive way.

We'd love your feedback on this feature as we continue to refine it. If you're an Octopus Cloud or Server customer, this is available from 2023.4.6612 onwards.

Happy deployments!
