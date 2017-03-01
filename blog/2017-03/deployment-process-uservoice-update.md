---
title: "Deployment process enhancements update"
description: A quick overview of some of the enhancements that have been shipped over the last few months to make configuring your deployment process a little bit easier
author: henrik.andersson@octopus.com
visibility: private
tags: 
 - New releases
---

As part of our [2017 roadmap](https://octopus.com/blog/roadmap-2017) we have set us a goal for UserVoice:

> By the end of 2017:
> - Octopus will have implemented all UserVoice items with over 200 votes

So to kick the year off the project modelling team (with contributions of a few inter-team members) we have implemented and shipped the following 3 UserVoice items since we started the new year that we hope should make configuring your deployment process a little bit easier:

- [Allow steps to be 'disabled' or 'inactive'](https://octopusdeploy.uservoice.com/forums/170787-general/suggestions/6324610-allow-steps-to-be-disabled-or-inactive) 
  - **649** votes
  - shipped in version `3.5.5`
- [Cloning of steps](https://octopusdeploy.uservoice.com/forums/170787-general/suggestions/6470009-cloning-of-steps) 
  - **282** votes
  - shipped in version `3.7.16`
- [Allow the Run Condition of a step to be based on a variable](https://octopusdeploy.uservoice.com/forums/170787-general/suggestions/6594872-allow-the-run-condition-of-a-step-to-be-based-on-a) 
  - **245** votes
  - shipped in version `3.7.13`

## Allow steps to be 'disabled' or 'inactive'
This feature was actually shipped in November 2016 but we wanted to mention it in this post as it was one of the highest customer requested features. 

What this feature allows you to do is to disable any step that might be causing issues with a deployment while configuring the project deployment process or you might just want to prevent the step from being run at deployment time temporarily. Previously you would either have to delete the step, assign the step to a role that doesn't do anything or skip the step at deployment time, not the cleanest or most user friendly solution!

Now, we've added an option to the context menu of the step that allows you to disable the step so that deployments can still be performed while ironing out any kinks with the new step.

To disable a step, select the `Disable` option from the step context menu
![](deployment-process-uservoice-update-disable-step.png)

Once a step has been disabled it will not be included in the deployment plan for any releases created while the step is disabled
![](deployment-process-uservoice-update-disabled-step.png)

To enable a disabled step again, simply select the `Enable` option from the step context menu 
![](deployment-process-uservoice-update-enable-step.png)

## Cloning of steps
This is a feature that was available in the BlueFin Chrome extension, but as not all organizations allow installation of browser extensions/plugins we decided to bring the feature into Octopus so that all our customers can take advantage of this functionality.

To clone a step, you simply select the `Clone` option from the step context menu
![](deployment-process-uservoice-update-clone-step.png)

This will copy the step you want cloned and add it below the step being cloned
![](deployment-process-uservoice-update-cloned-step.png)

## Allow the Run Condition of a step to be based on a variable
This feature allows you to tailor your deployment process **at runtime** by giving you the option to conditionally **run** or **skip** an action based on the **boolean** result of an [Octopus Variable Expression](http://docs.octopusdeploy.com/display/OD/Binding+syntax).

![](deployment-process-uservoice-update-variable-run-condition.png)

![](deployment-process-uservoice-update-variable-run-condition-selected.png)

To run a step based on the *truthy* value of a variable:

`#{VariableToCheck == "ValueToCheckAgainst"}`

To run a step based on the *falsy* value of a variable:

`#{VariableToCheck != "ValueToCheckAgainst"}`


To run a step based on the *truthy* value of a variable, and only if all previous steps were successful:

`#{unless Octopus.Deployment.Error}#{VariableToCheck}#{/unless}`

To run a step based on the *truthy* value of a variable, and if any of the previous steps have failed:

`#{if Octopus.Deployment.Error}#{VariableToCheck}#{/if}`