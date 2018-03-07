---
title: "Octopus March Release 2018.3"
description: What's new in Octopus 2018.3
author: tom.williams@octopus.com
visibility: private
metaImage: metaimage-shipping-2018-3.png
bannerImage: blogimage-shipping-2018-3.png
published: 2018-03-08
tags:
 - New Releases
---

Octopus 2018.3 brings a number of exciting new features including the [much requested feature to schedule recurring deployments](https://octopusdeploy.uservoice.com/forums/170787-general/suggestions/6599104-recurring-scheduled-deployments),

## In this post

!toc

## Release Tour

## Terraform Support

This release includes two new steps: Apply a Terraform template, and Destroy Terraform resources. These steps allow Terraform templates to be executed as part of an Octopus deployment project, with integrated support for variable substitution and output variable capturing. Refer to the [documentation](https://octopus.com/docs/deploying-applications/terraform-deployments) for more information on these new steps.

![Terraform Steps](terraform-steps.png "width=500")

## GitHub Feed Types

Adding GitHub as a feed type means that you can now deploy your external resources that don't require a dedicated pre-build step. Run scripts, templates or simple applications pulled directly from your source control by using tags to denote versions. Read more in our [Git Hub Feed documentation](https://octopus.com/docs/packaging-applications/package-repositories/github-feeds) for more information on using this new feed type.

## Highlight Messages and Artifacts

You can now write log messages that appear on the Task Summary tab of the Task page. These messages will also now appear blue and bolded within the task log and not be hidden under the `n additional lines not show` fold. We have added helpers to log at this level (`highlight`). Refer to the [documentation](https://octopus.com/docs/deploying-applications/custom-scripts#Customscripts-Logging) for the syntax to use in your script.

Another log level, `wait`, has been added to indicate that the deployment is waiting for something (for example a execution mutex). These messages also appear as a different color in the log. It is primarily used by Octopus internally, but you can also log at this level. In the future we plan on adding a timeline view, which will use log messages at this level to show when a deployment is paused.

Attachments will also now appear under the step they were collected in, but unlike the messages, they only appear once the step has completed due artifact collection only occurring at the end of the step.

![Highlights and Artifacts](highlights-and-artifacts.png "width=500")

## Upgrading

As usual [steps for upgrading Octopus Deploy](https://octopus.com/docs/administration/upgrading) apply. Please see the [release notes](https://octopus.com/downloads/compare?to=2018.3.0) for further information.

## Wrap up

That’s it for this month. Feel free leave us a comment and let us know what you think! Go forth and deploy!

## Dependent Libraries
Library Name | Version
--- | ---
Octopus.Client | 4.31.1
