---
title: ServiceNow Integration Improvements
description: Octopus updates ServiceNow Change Management integration
author: isaac.calligeros@octopus.com
visibility: public
published: 2024-04-24-1400
metaImage: blogimage-snow-2024.png
bannerImage: blogimage-snow-2024.png
bannerImageAlt: ServiceNow and Octopus Deploy logos connected by plugs with little stars around the connection.
isFeatured: false
tags:
- Product
- DevOps
- Integrations
---

Octopus added support for IT Service Management (ITSM), integrating with ServiceNow and Jira Service Management in 2022.3. Since then, we've received feedback highlighting points of friction and areas for improvement. We've revisited this feature, adding new functionality and some quality-of-life changes.

## Task log

Previously ITSM precondition checks would only show details on the Task Summary tab. We've now added logging to the process that creates the change request (CR). If the creation is successful this surfaces the CR number and a link to it. If unsuccessful, errors relating to the CR creation are now logged. These logging changes enhance visibility and troubleshooting during CR creation, particularly around permission and configuration issues.

![ITSM task logs](itsm-tasklog.png "width=500")

## Emergency changes

Octopus now supports creating emergency changes. To create an Emergency change request, you can now select the `Emergency Change` setting on the deployment creation page. Emergency changes are intended for scenarios such as resolving a major incident or implementing a security patch.

![ITSM deployment settings](itsm-deployment-settings.png "width=500")


## Transition to closed

For a deployment that creates its own CR, a per-project option, Auto Transition, will attempt to move the CR it created to the desired state when the deployment has been completed successfully. This option has now been extended to allow move a CR directly to closed.

## Populating CR fields through Octopus

To control the content of the CRs the variable `Octopus.ServiceNow.Field[snow_field]` can be set at the project level. These are contributed to the create CR body as a dictionary allowing any field to be set.

For example to set the `Assigned To` or `Short Description` fields you can use the following:

| Field | Variable | Example Value|
|--|--|--|
|Assigned To|Octopus.ServiceNow.Field[assigned_to]|beth.anglin|
|Short Description|Octopus.ServiceNow.Field[short_description]Custom Short Description with #{SomeVariable} #{Octopus.Deployment.Id}|

:::hint
Setting a `Short Description` will over-ride the auto generated Octopus description. Description matching means this will automatically progress the deployment unless the resolved description is unique. This can be done by including variables like the deployment or environment Id.
:::

:::hint
The expected ServiceNow value doesn't always align with the displayed value. In the case of `Assigned To` the value displayed is `Beth Anglin` but the expected value is the `User ID` in this case `beth.anglin`.
:::

For a full list of available fields and values, refer to the [ServiceNow docs](https://developer.servicenow.com/dev.do#!/reference/api/utah/rest/change-management-api).

## ITSM Providers menu 
We've also moved the ITSM Providers out of the `Project Settings` into their own seperate menu under the project links.


## Conclusion
Octopus continues to refine its ITSM capabilities based on ongoing user feedback and evolving needs. If you have any feedback or are interested in enabling ServiceNow or JSM please reach out.

[Register for the ServiceNow](https://octopusdeploy.typeform.com/servicenow-eap).
[Register for the Jira Service Management](https://octopusdeploy.typeform.com/jsm-eap)


## Learn more
- [ServiceNow documentation](https://octopus.com/docs/approvals/servicenow)

Happy deployments!