---
title: Request for Comments - Dynamic infrastructure
description: We're making changes to how Octopus manages dynamic infrastructure and we want your feedback.
author: rhys.parry@octopus.com
visibility: public
published: 2022-02-21-1400
metaImage: blogimage-feedback_2021_01.png
bannerImage: blogimage-feedback_2021_01.png
bannerImageAlt: Octopus salesperson at laptop with headset and icons representing customer feedback
isFeatured: false
tags:
 - Product
---

Cloud providers make it easy to provision new infrastructure. This means it's simple to spin up environments to test a new feature or perform a demo for a customer. Keeping your Octopus targets in sync, though, can be challenging as this infrastructure comes and goes.

Octopus currently provides a [dynamic infrastructure](https://octopus.com/docs/infrastructure/deployment-targets/dynamic-infrastructure) feature that lets you register targets during the deployment process. We give you the necessary pieces, and you glue them together to meet your requirements. This involves custom scripting, juggling output variables, and toggling settings.

This complexity has made it an expert use case, taking many attempts to put everything in the proper order. For many users, it's easier to register these targets manually. Because the functionality isn't clear, some won't even know there is another way.

We want to change all that. We want to leverage the way users already interact with their cloud providers and Octopus, to create a solution that feels unobtrusive and intuitive.

## Our solution

We want Octopus to find and register targets for your deployments. We've broken out the key pieces to make it all work.

### Tag-driven target discovery

Instead of directly registering deployment targets in Octopus, we'll discover targets when needed. 

You'll apply tags to your cloud resources to represent your intent, and Octopus will discover and register these targets when executing your deployment process.

First, your resource will require two tags: `octopus-environment` and `octopus-role` to indicate the environment and role, respectively. You'll also be able to specify tags to restrict discovery by project, space, and tenant.

### Authentication context

To discover targets, Octopus will need to authenticate with the appropriate cloud provider. You'll be able to provide this context using Octopus variables. For example, you'll be able to configure a variable named `Octopus.Azure.Account` to reference the Azure account to use for discovery. We'll link the account from the authentication context during discovery to the target by default.

We like variables for this purpose because they have well-established scoping behavior that can be used to configure different accounts for each applicable scope. So, for example, you could scope by the environment, ensuring test and production environments use different authentication contexts.

### Target lifecycle

A key benefit of the cloud is how easy it is to spin up new environments and tear them down when you're done with them. Octopus will keep track of these discovered targets and clean them up if they no longer exist. If, for some reason, they come back, Octopus will re-discover them for the next deployment.

## FAQ

### Will all my cloud resources become targets in Octopus?

No. Only resources that map to Octopus targets used by our built-in steps will be subject to discovery. Octopus will also need the resource to have the tags `octopus-role` and `octopus-environment` set. Octopus will ignore any resource without these tags during discovery.

### Will the traditional way of working with deployment targets still work?

Existing custom scripts will continue to work, and the PowerShell commandlets for creating new targets will remain unchanged. Migrating to this new method will only involve:

- Setting the appropriate tags on your cloud resources
- Removing your existing script steps
- Configuring the authentication context variable

Targets can still be manually registered.

### Will all my targets be automatically deleted if they're not found?

Only targets created due to discovery through this mechanism will be automatically deleted. Targets created through the UI or the existing commandlets will behave as they do now.

### What if I need to use different accounts for deployment and discovery?

You'll be able to specify the extra details Octopus needs for configuring your targets by setting other tags. For example, to use a different account when deploying to a resource, you'll be able to specify the `octopus-account` tag to reference the desired account in Octopus.

### What targets will be supported?

Initially, we'll be working to support Azure Web Apps, Service Fabric, and AWS ECS clusters. Then, as we expand our support for additional cloud targets in the coming year, we'll ensure they're ready to go for dynamic infrastructure.

We plan to follow this with support for discovering Kubernetes containers across AWS, Azure, and Google Cloud.

### Will AWS instance/assumed roles be supported?

Octopus currently provides a wide range of options for authenticating with AWS. In addition to an account using an access key and secret key, we support assuming an IAM role and using the role assigned to a Worker.

We're still investigating how to best support this and juggle the Worker Pool dependency when relying on instance roles.

## We want your feedback

We're actively developing this feature and would love to incorporate your feedback. We've created a [Github issue](https://github.com/OctopusDeploy/StepsFeedback/issues/8) to capture your comments.

Specifically, we want to know:

- About edge cases we might not have anticipated in regards to tagging
- Will this solution fit your use case? 
- Are there other dynamic infrastructure challenges you'd like to see Octopus address?

This feedback will help us deliver the best solution we can.

<span><a class="btn btn-success" href="https://github.com/OctopusDeploy/StepsFeedback/issues/8">Provide feedback</a></span>

## Conclusion

In summary, our plan for improving dynamic infrastructure involves:

- Defining tags to be used on cloud resources to describe intent during discovery
- Providing an authentication context via variables with well-known names
- Registering targets during the deployment process
- Cleaning up old targets that no longer exist in the cloud

Thanks for reading. We hope you're as excited as we are about the potential this functionality will unlock.

Any [feedback](https://github.com/OctopusDeploy/StepsFeedback/issues/8) you have is greatly appreciated.

Happy deployments!
