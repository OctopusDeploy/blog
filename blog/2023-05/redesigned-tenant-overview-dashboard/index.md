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
  - Product
  - Multi-Tenancy
---

Tenants in Octopus are the standard way for you to create customer-specific deployment pipelines without duplicating project configurations. Tenants help you deploy multiple instances of an application for each of your customers. 

We received feedback that we could improve our Tenants overview dashboard, to make it easier to administer manage, and view hundreds or thousands of tenants.

We addressed this by redesigning the dashboard.

The new Tenants overview dashboard gives you better overall visibility of your tenants and tenant-related items. This includes projects, environments, and tenant tags. The new dashboard is also fast and performant under large data loads. You can easily manage and administer thousands of tenants, knowing the dashboard won't hang, crash, or slow down.

![blogimage-redesignedtenantoverviewdashboard](https://github.com/OctopusDeploy/blog/assets/102109515/4dd18180-cee1-4c04-8d25-490e082f53cc)

In this post, I show you how the new redesigned Tenants overview dashboard works.

## Using a new data table

Before this redesign, the Tenants overview dashboard looked like this:

![blogimage-oldtenantoverviewdashboard](https://github.com/OctopusDeploy/blog/assets/102109515/4aa82837-6f07-40c7-bdb1-5743eb61e5d4)

Some customers found it challenging to navigate through this card-style format to see lots of data at the same time. We solved this by displaying the information in a data table style format.

![blogimage-newdatastyletableformat](https://github.com/OctopusDeploy/blog/assets/102109515/8b585e3d-ba28-4686-a950-f645b9c171c8)

Visualizing the information this way preserves existing functionality for viewing tenants. It also lets you see other tenant-related information that you couldn't see with the card-style format, namely:

- The name and number of tenant tags associated with each tenant
- The projects associated with each tenant
- The environments associated with those projects

## Expandable rows to view additional information

![blogimage-expandablerows](https://github.com/OctopusDeploy/blog/assets/102109515/6ac8fa55-f8e1-47ca-af5c-b9264d5cd211)

The data table format allows Octopus to provide more information at a glance. By default, the table reveals high-level information about tenant tags, projects, and environments. It then hides the rest of the information in an expandable row.

You can open the expandable row if you want more detail about those items. This displays all information about tenant tags, projects, and environments for that tenant. You can see all the related information or retract the row to keep your view clean and organized.

## Pagination and results controls

![blogimage-paginationcontrol](https://github.com/OctopusDeploy/blog/assets/102109515/0529853f-21d2-4b8f-b010-bb84cd764367)

![blogimage-resultscontrol](https://github.com/OctopusDeploy/blog/assets/102109515/1b81a487-4b9a-4653-a55f-444150b0a3be)

The Tenants overview dashboard now has additional pagination and results limitation and expansion controls. You can choose to see 30, 50, or 100 results. Pagination follows the option selected for results limitation. For example, if you select 50 results, each dashboard page shows up to 50 results before you need to click through to the next page.

The pagination and results limitation means the new dashboard suits different use cases. You can see all data or just some, depending on your preferences and needs.

## Maintaining existing filtering and name search function

![blogimage-firstnamesearch](https://github.com/OctopusDeploy/blog/assets/102109515/77718673-c7ba-48d8-9d19-77cb2fc9039a)

We kept the advanced filters on the dashboard. You can still find the **Filter by name** search bar on the top left-hand side of the page. Use the search bar to type in a tenant's name to narrow down the results and find the tenant you're looking for.

![blogimage-advancedfilters](https://github.com/OctopusDeploy/blog/assets/102109515/8b6e444b-adfc-4e4f-a33d-a4072932de95)

You can also use these advanced filters with the new dashboard to search for a particular tenant or other tenant-related items.

## Performant under large data loads

Finally, the new dashboard is performant and fast under large data loads. This is the case even if you have hundreds or thousands of tenants to view, manage, and administer. The dashboard is more responsive and reliable, without long loading times or sluggish UI performance.

## Conclusion

The new Tenants overview dashboard provides greater visibility and better tools to manage and administer your tenants at scale. 

We'd love feedback on this feature while we continue to refine it. If you're an Octopus Cloud customer or running Octopus server version 2023.2.10424 and above, it's available now as an early access preview (EAP). 

<span><a class="btn btn-success" href="https://octopusdeploy.typeform.com/to/CxkblnbR">Share your feedback</a></span>

Happy deployments!
