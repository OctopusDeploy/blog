---
title: The difference between day-0, day-1, and day-2 operations
description: We break down the difference between the 3 major phases of operations in DevOps.
author: andrew.corrigan@octopus.com
visibility: private
published: 2023-12-11-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - DevOps
  - Runbooks
---

Operations teams have always been important in any organization that uses tech to do business. Thanks to [DevOps](https://octopus.com/devops/) (the best methodology we know to improve processes between developers and operations), we can see just how important they are when that business *is* tech.

Simply put, operations is a function as essential to software delivery as development. We all know software can't exist without developers. Unless it's a small project, software also can't run at all, or for as long as it should, without those who set up, monitor, and maintain the environments it lives on.

Despite the unifying nature of DevOps, it feels like the software industry knows way more about development practices than operations practices. So, let's do something about that!

In this post, we explore the 3 core phases of operations in software development: day-0, day-1, and day-2.

## Day-0 operations

It's weird to start a numbered list of things with 'zero,' I'll give you that. But, it makes sense given day-0 operations usually start before there's anything to, well, *operate*.

Which is to say, day-0 is all about *planning*.

Whether it's a brand new product or new features, all interested parties meet to discover what's needed to make sure the software performs as expected.

Day-0 usually helps answer questions like:

- Does the software architecture change at all?
- Do we need new infrastructure or changes to existing infrastructure?
- Do we need new hardware to make the changes?
- Is there a need for new tooling in the pipeline?
- Does the change affect databases? When do we back up or restore them?
- Is there anything new we need to monitor?
- Do we have any hard limits around finances? Can we afford how much this will cost, or do we need to scale back our ambitions?

The bigger your organization, the more questions you probably need to answer. As your software grows more complex, the answers may take longer to find.

Though the industry delivers many applications through the cloud where infrastructure setup can happen in a deployment process, you may find some bleed between day-0 and day-1 operations.

For example, setup would start before deploying the application if using self-hosted servers to run software. After all, you can't deploy to targets that don't exist yet.

Plus, if you need new tools in the pipeline, they'll need to be in place for deployment too.

## Day-1 operations

Day-1 operations is where ideas become reality. The development team completed their work and committed their changes. Packages now work their way through the deployment pipeline built and managed by the operations team.

That's right, day-1 is *all* about deployments.

Day-1 operations tasks include:

- Setup or creation of new infrastructure if it doesn't already exist
- Pre-deployment database backups and post-deployment restores
- Starting the deployment (if the pipeline isn't fully automated)
- Stopping the deployment and starting rollbacks if there are major problems

How deployments happen can differ between organizations, teams, and even projects. Manual deployments for small projects aren't uncommon, but they're not sustainable long term.

And hey, big or small, deployments done badly can become a huge cause of anxiety for many in an organization. The reason is that deployments are no longer only a case of running an executable on a server somewhere.

Deployment automation can help remove that anxiety in almost every single case. Best practice methodologies, like DevOps and Continuous Delivery, recommend automating and improving your deployment process as much as possible.

Like day-1 operations, Octopus is also *all* about deployments, but we're specifically about deployment *automation*. We believe deployments should be smooth-going and celebrated rather than feared.

Hold that thought for now - we'll come back to that later.

## Day-2 operations

Day-2 for operations is the Boxing Day of software development. The fun stuff is over. Celebrations over a 'successful delivery' are behind you. It's time to embrace reality for a while, at least until you start all over again.

If following Continuous Integration and [Continuous Delivery](https://octopus.com/devops/continuous-delivery/) well, however, that will happen way more often than once a year. In fact, it's a process that should happen constantly.

Yes, day-2 is the stuff that happens *after* deployment.

Day-2 operations tasks include:

- Monitoring the application to prevent and respond to problems
- Gathering data that will help improve the pipeline and its processes in the next sprint
- Performing routine maintenance like backups and restores
- Restoring service if infrastructure fails, through spinning up new infrastructure or doing rollbacks

You can sum up day-2 operations as operations 'business as usual' tasks. It's the phase that keeps your product ticking over so your customers can enjoy it when they need it.

Fret not - another day-0 is just on the horizon.

## Conclusion

Weirdly, there's a perception that day-2 is where all an operations team's work takes place (and if your organization has that perception, [it might be a sign your teams don't collaborate as well as they should](https://octopus.com/devops/culture/)). Operations should be ever-present throughout the entire deployment pipeline.

As we talked about, operations teams can help keep ideas in check. They're important in tool discovery and finding out what developers need to make those ideas real. They're active during the deployment process and, finally, they're vital in the application's ongoing performance.

Octopus believes good deployment tools should recognize and cater for the roles of operations in the pipeline. [Octopus's built-in Runbooks feature](https://octopus.com/features/runbooks) can help organizations automate many of the tasks we mentioned in this post, routine or otherwise.

Happy deployments!