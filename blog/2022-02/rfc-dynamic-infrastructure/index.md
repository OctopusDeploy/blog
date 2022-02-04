---
title: RFC - Dynamic Infrastructure
description: We're making changes to how Octopus manages dynamic infrastructure, and want your feedback.
author: rhys.parry@octopus.com
visibility: public
published: 2021-02-08-1400
metaImage: blogimage-feedback_2021_01.png
bannerImage: blogimage-feedback_2021_01.png
bannerImageAlt: Octopus salesperson at laptop with headset and icons representing customer feedback
tags:
 - Product
---

Cloud providers have made it extremely easy to provision new infrastructure. This has made it easy to temporarily spin up entire environments to test a new feature or perform a demo for a customer. However, keeping your Octopus targets in sync can be challenging as this infrastructure comes and goes.

Octopus currently provides a [dynamic infrastructure](https://octopus.com/docs/infrastructure/deployment-targets/dynamic-infrastructure) feature that allows registering targets during the deployment process. We give you the necessary pieces, relying on you to glue them together to meet your requirements. This involves custom scripting, juggling output variables and toggling a few settings.

This complexity has led to this becoming an expert use case, typically taking multiple attempts to put everything in the proper order. For many users, it's easier to register these targets manually. Because the functionality isn't apparent, some won't even know there is another way.

We want to change all that. We want to leverage the way users already interact with their cloud providers and Octopus to create a solution that feels unobtrusive and intuitive.

## Our solution

We want Octopus to do the work of finding and registering targets for your deployments. We've broken out these key pieces to make it all work.

### Tag-driven target discovery

Instead of directly registering deployment targets in Octopus, we'll discover targets when needed. You apply tags to your cloud resources to represent your intent, and Octopus will discover and register these targets when executing your deployment process.

First, your resource will require two tags: `octopus-environment` and `octopus-role` to indicate the environment and role, respectively. You can also specify tags to restrict discovery by project, space and tenant.

### Authentication context

To discover targets, Octopus needs to authenticate with the appropriate cloud provider. You can provide this context using Octopus variables. For example, you can configure a variable named `Octopus.Azure.Account` to reference the Azure Account you would like to use for discovery. We'll link the account from the authentication context during discovery to the target by default.

We chose to use variables for this purpose because they have well-established scoping behaviour that can be used to configure different accounts for each applicable scope. So, for example, you could scope by the environment, ensuring Test and Production environments use different authentication contexts.

### Target lifecycle

One of the key benefits of the cloud is how easy it is to spin up new environments and tear them down when you are done with them. Octopus will keep track of these discovered targets and clean them up if they no longer exist. If, for some reason, they come back, Octopus will happily re-discover them for the next deployment.

## FAQ

### Will all my cloud resources become targets in Octopus?

No. Only resources that map to Octopus targets used by our built-in steps will be subject to discovery. Octopus will also require the resource have the tags `octopus-role` and `octopus-environment` set. Octopus will ignore any resource without these tags during discovery.

### Will the traditional way of working with deployment targets still work?

Existing custom scripts will continue to work, and the PowerShell commandlets for creating new targets will remain unchanged. Migrating to this new method will only involve setting the appropriate tags on your cloud resources, removing your existing script steps and configuring the authentication context variable.

Targets can still be manually registered.

### Will all my targets be automatically deleted if they are not found?

Only targets created due to discovery through this mechanism will be subject to automatic deletion. Targets created through the UI or the existing commandlets will behave as they now.

### What if I need to use different accounts for deployment and discovery?

You'll be able to specify additional details Octopus might require for configuring your targets by setting other tags. For example, to use a different account when deploying to a resource, you can specify the `octopus-account` tag to reference the desired account in Octopus.

### What targets will be supported?

Initially, we'll be working to support Azure Web Apps, Service Fabric and AWS ECS Clusters. Then, as we expand our support for additional cloud targets in the coming year, we will ensure they are ready to go for dynamic infrastructure.

We expect to follow this with support for discovering Kubernetes containers across AWS, Azure and Google Cloud.

### Will AWS instance/assumed roles be supported?

Octopus currently provides a wide range of options for authenticating with AWS. In addition to an account using an access key and secret key, we support assuming an IAM role and using the role assigned to a worker.

We are still investigating how to best support this and juggle the worker pool dependency when relying on instance roles.

## We want your feedback

We're actively developing this feature, and we'd love to incorporate your feedback. We have created a [Github issue](https://github.com/OctopusDeploy/StepsFeedback/issues/8) to capture the discussion.

Specifically, we want to know:

- About edge cases we might not have anticipated in regards to tagging.
- Will this fit your use case, or are there other dynamic infrastructure challenges you'd like to see Octopus address?

This feedback will help us deliver the best solution we can.

## Conclusion

In summary, our plan for improving dynamic infrastructure involves:

- Defining tags to be used on cloud resources to describe intent during discovery.
- Providing an authentication context via variables with well-known names.
- Registering targets during the deployment process.
- Cleaning up old targets that no longer exist in the cloud.

Thanks for reading this post. We hope you are as excited about the potential this functionality will unlock.

Any [feedback](https://github.com/OctopusDeploy/StepsFeedback/issues/8) you have is greatly appreciated.

Happy deployments!
