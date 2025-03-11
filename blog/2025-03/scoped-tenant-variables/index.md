---
title: Scoped Tenant Variables
description: Learn about new scoping capabilities of Tenant Variables.
author: susan.pan@octopus.com
visibility: public
published: 2025-03-24-0900
metaImage: blogimage-variable-tenants-ui-dashboard-2024.png
bannerImage: blogimage-variable-tenants-ui-dashboard-2024.png
bannerImageAlt: Graphic image of woman viewing scoped tenant variables page.
isFeatured: false
tags: 
  - Product
  - Tenant Variables
  - Variables
---

I'm here to announce exciting new improvements to tenant variables - scoped tenant variables! 

[Tenant variables](https://octopus.com/docs/tenants/tenant-variables) are great for managing variables both within a tenanted project and across all project environments connected to a tenant. However, there isn't an easy way to manage variables across all projects connected to a tenant. Scoped tenant variables aims to solve this.

Assigning environment scopes to your tenant variables will make variable management across tenant projects so much simpler. In this post I will walk you through how to scope tenant variables and discuss the current integration capabilities.

These updates are now available to our Cloud customers and will be available to self-hosted customers from v2025.2.

## Why should I scope tenant variables?

Previously, there were both constraints and differences in how tenant variables were managed with respect to environments. 
- Tenant project variables were required to have one value per project environment. 
- Tenant common variables were limited to one value across all environments connected to a tenant. 

Now, both project variables and common variables can be scoped to environments as needed. This means they can be unscoped (variable applies to all connected environments), singly-scoped or multi-scoped. 

Adding the ability to assign scopes to tenant variables creates more simplicity and flexibility with variable management. For example, to assign different variable values for different environments across all tenant projects, customers had to create a tenant for each environment. A common variable could then be set for each tenant. Now, simply set multiple common variables with the appropriate environment scope. 

This is particularly useful if your non-production environments all share one value, while production has a separate value; maintenance of variables becomes more straightforward as values only need to be updated once.

## How to assign scopes to your tenant variables
Tenant variables can be set both from the projects page and from the tenants page. I will show you how to set tenant common variables from the project page. 

Create a tenant with a connected project and connected environments. Create a Variableset with a variable template and connect it to the tenanted project. 

- Select the tenant variables page from the sidebar and click on the common templates tab.
![Screenshot of common tenant variables tab on tenant variables page.](scoped-tenant-vars-view.png)

- To add a new tenant variable, select the overflow menu and click 'Add'. Type in a value and select the environments in the 'scope' column. Click 'Save'.
![Screenshot of adding a new tenant variable on tenant variables page.](scoped-tenant-vars-add-new.png)

- The page should now show the new tenant variable values and scopes.
![Screenshot showing new scoped tenant variables on tenant variables page.](scoped-tenant-vars.png)

Unscoped variables will apply to all environments with no explicitly scoped variables. [See our docs for more information on scoping variables](https://octopus.com/docs/projects/variables/getting-started#scoping-variables)
 
### Migrating existing tenant variables
Existing tenant variables will automatically be migrated over to the new scoped tenant variables when upgraded to 2025.2. The migration will scope tenant project variables to the previous single scope and common variables to be unscoped. This ensures your deployments can proceed without any changes required. 
. Once your instance detects scoped tenant variables being used, it will prevent the use of the old tenant variable endpoint. 

:::info
The endpoint ```tenants/{tenantId}/tenantvariables``` will be deprecated from 2026.2.
:::

We have introduced 2 new endpoints for GET/UPDATE requests for scoped tenant variables. These will replace the use of the endpoint above: 
- ```tenants/{tenantId}/projectvariables```
- ```tenants/{tenantId}/commonvariables```
 
## Automation and integration
Scoped tenant variables are now supported through our dotnet, typescript and go clients. [See our docs for more information on our API clients](https://octopus.com/docs/octopus-rest-api/getting-started#api-clients)

Octopus CLI will also support the use of scoped tenant variables. As our CLI matches tenant variables on the variable's name and environment, the tenant variable update function currently doesn't support changing variable scopes of existing tenant variables. To select the correct tenant variable, the exact scope must be specified. 

Our terraform provider does not currently support the new scoped tenant variables, however, plans are in place to add support in the near future.

## Conclusion
Scoped tenant variables give you finer control of your tenant variables. Now, you can simply assign a scope to your tenant variables and watch the magic happen.

To get started or learn more, you can read our [tenant variable documentation.](https://octopus.com/docs/tenants/tenant-variables)

Happy deployments!