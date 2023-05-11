---
title: Redesigned Tenants overview dashboard
description: Learn how we redesigned our Tenants overview dashboard to make it easier to view and manage thousands of tenants.
author: ian.khor@octopus.com
visibility: private
published: 2022-05-24-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - tag
---

Tenants in Octopus are the standard way for you to create customer-specific deployment pipelines without duplicating project configurations. Tenants help you deploy multiple instances of an application for each of your customers. 

We received feedback that we could improve the dashboard, to make it easier to administer manage, and view hundreds or thousands of tenants.

We addressed this by redesigning the tenant overview dashboard.

The new dashboard gives you better overall visibility of your tenants and tenant-related items. This includes projects, environments, and tenant tags. The new dashboard is also fast and performant under large data loads. You can easily manage and administer hundreds or thousands of tenants, knowing the dashboard won't hang, crash, or slow down.

![Screenshot 2023-05-11 at 3 36 14 pm](https://github.com/OctopusDeploy/blog/assets/102109515/636e1c21-f7bb-479d-92ce-8a05f9ed847d)

In this post, I show you how the new redesigned Tenants overview dashboard works.

## **Using a new data table**

Previously, the tenant overview dashboard looked something like this:
![image](https://github.com/OctopusDeploy/blog/assets/102109515/c7d09719-60d0-41c3-a35f-48969e39bf10)

Although suitable for some use cases, customers would find it difficult to navigate through this card-style format when wanting to see large amounts of data displayed at the same time. This problem is solved through the new tenant dashboard, which displays the information in a data table style format.
![Screenshot 2023-05-11 at 3 38 34 pm](https://github.com/OctopusDeploy/blog/assets/102109515/6053709a-3e31-4fb0-8653-0e65bfe22936)

Visualizing the information in this manner not only preserves existing functionality around viewing their tenants, but gives users the ability to see other tenant related information that they previously were not able to using the old card-style format, namely:

- The name & number of tenant tags associated with each tenant
- The projects associated with each tenant; and
- The environments associated with those projects

## **Expandable Rows To View Additional Information**
![Screenshot 2023-05-11 at 3 39 47 pm](https://github.com/OctopusDeploy/blog/assets/102109515/a6998b51-bab9-49b4-ad1a-1ff29118b28c)
Another advantage of the data table format for administering, viewing and managing tenants is that it allows Octopus to provide more information related to a tenant that are important for customers to know at first glance. For example, by default, the table provides information about tenant tags, projects and environments up to a certain amount, before hiding the rest of the information in an expandable row.

For customers that do want to see more information related to those items, they can simply open the expandable row, which will display all of the tenant tag, project and environment information related to that tenant. This not only allows a customer to see all the related information if required but also allows them to retract the row when needed, keeping the tenants and tenant related information neat and organised.

## **Pagination & Results Controls**
![Screenshot 2023-05-11 at 3 40 44 pm](https://github.com/OctopusDeploy/blog/assets/102109515/dfaf8b23-6c09-4a42-8db0-d2ed8fa8bc7f)

Other than the increased display of information, the new tenant overview dashboard now has additional pagination and results limitation/expansion controls. Result limitation/expansion options range range from 30, 50 & 100 results, and pagination follows the option selected for results limitation. For example, if the number of results to be shown is ‘30’, then each ‘page’ of the dashboard will show up to 30 results before having to paginate over to the next page.

The combination of both pagination and results limitation means the new tenant overview dashboard can suit a variety of different customer use cases when it comes to visualising tenants and tenant related data and whether they want to see all of the data at once or just some of the data at the time, depending on the customer preference and use case.

## **Performant Under Large Data Loads**
Finally, the new tenant overview dashboard is performant and speedy under large data loads, particularly when a customer has 100s or 1000s of tenants which they want to view, manage and administer using the tenant overview dashboard. Customers can be reasonably assured that the dashboard will be responsive and reliable, and will avoid any long loading times or sluggish UI performance when using this for large sets of data or information relating to their tenants.

## Conclusion

The new tenant overview dashboard provides you greater visibility and better tools to manage and administer your tenants at scale. We’d love feedback on this feature while we continue to refine it. If you’re an Octopus Cloud customer, or running Octopus server version 2023.2.10424 and above, you will receive early access preview (EAP) immediately. Please feel free to provide feedback using this [link](https://octopusdeploy.typeform.com/to/CxkblnbR).

Happy Deployments!
