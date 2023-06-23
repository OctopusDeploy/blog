---
title: Introducing Execution Timeouts
description: Learn about our new Execution Timeouts feature to handle unexpectedly long-running processes
author: isaac.calligeros@octopus.com
visibility: public
published: 2023-06-26-1200
metaImage: execution-timeout-logo.png
bannerImage: execution-timeout-logo.png
bannerImageAlt: TODO
isFeatured: false
tags:
  - Product
---

We've received [user voice](https://octopusdeploy.uservoice.com/forums/170787-product-feedback/suggestions/6396476-add-timeout-support-for-individual-steps-and-overa) feedback requesting Execution Timeouts, a feature to handle hung deployment processes. We're excited to announce that Octopus 2023.3 introduces Execution Timeouts, a way to configure automatic step cancellations. Some deployment processes such as Azure FTP connections and CloudFormation updates include long-running actions. These can last for hours or even hang indefinitely. With Execution Timeouts, you can now set a time limit in minutes within steps a step configuration. When this timeout lapses, the action fails, and the deployment process continues. Paired with auto-retries, this increases the likelihood of successful deployments.

Watch the video below to see this new feature in action:

<iframe width="560" height="315" src="TODO" title="YouTube video player" frameborder="0" allow="accelerometer; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

## How do Execution Timeouts work?

You can enable Execution Timeouts on individual steps found in the **Conditions** section of each step, as shown below:

![Execution Timeouts configuration on a step.](execution-timeout-ui.png)

Execution Timeouts are configured in minutes. An unset value or zero won't timeout. It's important to note that Execution Timeouts encompass all processes involved in step execution. This includes connecting to a target, worker leasing, bootstrapper scripts, execution container start-up, and package cache clean-ups. We recommend setting a slightly longer timeout than expected, in most cases, an additional minute should account for this.

### Are there any steps I can't use Execution Timeouts for?

You can use Execution Timeouts on all steps except for:

- **Send an Email** step
- **Manual Intervention** step
- **Health Check** step
- **Deploy a Release** step
- **Kubernetes** steps, these have a kubernetes specific implementation in the step configuration.

### How are Execution Timeouts configured?

At the time of this blog Execution Timeouts are an EAP feature behind a feature toggle. If you're a cloud customer and looking to try out this feature please send us an email at [support@octopus.com](mailto:support@octopus.com). Once enabled Execution Timeouts can be configured manually through the step UI, or through the octopus variable `Octopus.Action[stepName].ExecutionTimeout.Minutes`. To avoid warning logs about overriding variables you'll need to also set the `OctopusSuppressDuplicateVariableWarning` variable to true.

### Can I set Execution Timeouts to a default value for all my steps?

No. You must configure Execution Timeouts on each step in each deployment process.

### Do Execution Timeouts trigger at all stages of a deployment process?

No, Execution Timeouts won't trigger during package acquisition.

## Conclusion

To help improve the success rate and automation of deployments, you can now add Execution Timeouts. This lets you configure unpredictable long-running processes to fail early. Combined with automatic retries this feature can be used to improve deployment success rates.

## Feedback

We'd love to hear [any feedback](https://oc.to/ActionExecutionTimeOutFeedbackForm) to help us improve this feature.

Happy deployments!