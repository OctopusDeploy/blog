---
title: Introducing step retries
description: Step retries is a new feature to combat transient connectivity issues and improve deployment success rates
author: michelle.obrien@octopus.com
visibility: public
published: 2099-06-01-1200
metaImage: 
bannerImage: 
bannerImageAlt:
isFeatured: false
tags:
  - Product
---


Octopus 2023.2 introduces step retries, a new feature allowing the automation of retries within steps. This feature is beneficial when dealing with steps that commonly fail due to temporary or transient errors during deployment.

The video below demonstrates an in-depth dive into the new feature:
<iframe width="560" height="315" src="https://youtu.be/2KzwjpdZz70" frameborder="0" allowfullscreen></iframe>

## How do auto-retries work?
Users can enable retries on individual steps found within the ‘Conditions’ section of each step as below:
![step retries](blogimage-stepautoretries-2023.png)

When you enable retries for a step, and an action within that step fails, an automatic retry will be initiated specifically from the failed action (note that the entire step does not retry from the beginning). In the event of a subsequent failure, there will be a 15-second delay before the action retries. A final retry with a 15-second delay will also take place. If any of these retries are successful, the deployment will proceed. However, if all three retries are unsuccessful, the step will fail, or guided failure mode will be activated if enabled.

## FAQ

### Is this available on all steps?
This is available on all steps except for _Send an Email_, _Manual Intervention_, _Health Check_ and _Deploy a Release_ steps.

### Can I customize the number of retries?
Not at this stage, but please feel free to provide feedback.

### Can I set retries to default to ‘on’ for all my steps?
You must configure retries on each step you want to retry within each deployment process.

### How do retries work with guided failure mode?
If enabled, Guided Failure Mode will kick in if all three retries are unsuccessful.

## Feedback
We'd love to hear [any feedback](https://octopusdeploy.typeform.com/to/UOObqaxV) to help us improve this feature.