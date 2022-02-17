---
title: Best practices for Octopus Spaces
description: Find out how to use Spaces effectively.
author: steve.fenton@octopus.com
visibility: private
published: 9999-01-01
metaImage:  blogimage-spaces.png
bannerImage: blogimage-spaces.png
tags:
 - Product
 - Spaces
---

Over the years at Octopus, we've always been pleased to see our customers expand their use of Octopus Deploy to include new projects and teams. In the early days, it was hard to imagine this could ever pose any kind of problem. However, there comes a point when having too many projects in Octopus can become a distraction, particularly for a team that only works on a handful of those projects.

The Octopus Spaces feature helps you organize and secure your projects, environments, and infrastructure and control which team members can access them. They are a useful alternative to installing multiple Octopus Deploy instances.

In this post, you'll learn how to use spaces effectively to organize your deployments.

## An overview of spaces

Spaces are partitions that create hard walls in your Octopus Server. Deployment resources assigned to a space cannot be seen or used from another space. You can use spaces instead of creating multiple instances of Octopus Deploy, with each space being self-contained.

The following items are scoped to a space:

- Environments
- Lifecycles
- Projects
- Variable sets
- Deployment targets
- Tenants

These space-scoped items cannot be accessed from other spaces.

Teams are a special case. When you create a team, you can choose whether to scope them to a single space or have the team span all spaces.

All events are written to the system-level audit log, which you can filter per space.

An Octopus Administrator can give full responsibility for managing each space to a Space Manager, which can reduce the workload for the administrator.

The [administration guide for Spaces](https://octopus.com/docs/administration/spaces) has instructions for managing spaces.

## The benefits of spaces

There are two primary use cases for spaces:

- Organising deployment resources
- Controlling access to these resources

You might choose to use spaces for either or both of these reasons.

### Organising deployment resources

Your Octopus Deploy Dashboard displays a row for each project and a column for each phase in your lifecycles. The dashboard gets taller when you add more projects and wider as you create more phases. If you find your dashboard overwhelming, moving projects into spaces will clean it up and reduce how much you have to scroll.

You will get the same benefits across all screens in the space and when choosing resources from a list, such as editing a process step, as only items from the current space are shown.

Spaces allow you to limit the growth in each area by grouping related resources together.

### Controlling access to deployment resources

Before we introduced spaces, you had some control of the visibility of projects using teams with scoped roles. However, this could become difficult to manage. It was hard to stop items like deployment targets from being re-used by new projects.

Spaces give you a convenient way to control access to a group of related resources without complicated permissions. You can grant a team member full or read-only access to the spaces they need, and they can then switch between them quickly using the space switcher.

![The space switcher appears in the top-left of the Octopus Deploy navigation bar](space-switcher.jpg)

## How to design your spaces

A space represents a logical group of applications. If you have several closely related components, they are likely to share some variables in a variable set or be deployed to the same infrastructure. These factors will naturally guide whether they should be grouped into a space or kept in separate spaces to prevent them from becoming related in undesirable ways.

It may seem like a good idea to create a space for each team in your organization, but this is not always the best design to follow. In particular, this may lead you to split items into too many spaces in situations where multiple teams contribute to the same application.

Instead, use one of the following dimensions to design your spaces:

- Clients
- Application groups
- Application audiences
- Company divisions

Read on for more information about each of these options.

:::hint
You can use the [export/import project feature](https://octopus.com/docs/projects/export-import) to move projects to a new space.
:::

### Client

If you are an agency managing deployments for multiple clients, using a space per client ensures no data will be shared between customers. All deployment targets, projects, variables, and lifecycles will be exclusive to the customer space.

You may decide to sub-divide a client into multiple spaces using one of the other options.

### Application groups

An application group is a set of related components that are likely to be deployed to the same location.

Application groups are ideal for organizing spaces because similar design considerations apply to spaces that will have influenced your application design. For example, you might group the components of a content management system (CMS) within one space and the components of a billing system into a second space.

You may find that this design for spaces matches how you have organized your teams. It is fine to land on this solution if you have done so by considering your application groups.

If you have more than one team contributing to an application group, you should maintain a space that aligns with the software, not the teams. You can give both teams access to the space, and both can see a whole-system view of deployments.

### Application audiences

A less granular approach is to have separate spaces that match intended audiences, for example, internal and public-facing applications. This approach will clean up the public-facing space by moving internal resources into a different space.

This design can quickly improve the information for your public-facing space and may serve as a first step toward splitting resources into spaces for each application suite.

### Company divisions

If your company is organized into divisions that develop independent applications, this is likely to provide a natural design for spaces. For example, if the company has divisions that offer software to different industries, each division could have a separate space with a dedicated space manager.

With their own space manager, each division would be self-sufficient in managing its space without cluttering Octopus Deploy for other divisions.

## Useful design indicators

The ideal scenario is that applications within a space are independent, deployed to dedicated targets, and have an autonomous team responsible for them. While you might not find yourself in this perfect situation, it provides a helpful guide when deciding how to design your spaces.

If you deploy multiple applications to the same deployment targets, you should keep the deployments within the same space. It is possible to set up the same deployment target in more than one space using a listening tentacle. However, this complicates the permissions, and team members won't see all the deployments targeting the shared infrastructure.

If there are strong reasons to split the deployments into multiple spaces, those same reasons are likely to mean you should have different deployment targets, too.

## What to avoid

You should avoid using spaces for each environment, as you will need to duplicate the process in each space. It would not be easy to keep the process consistent in each space when you make changes and you won't benefit from release snapshots. Release snapshots make sure the same version of the package, variables, and process is used throughout environments, and this improves the reliability of your deployments.

You should also avoid using spaces where it is more appropriate to use [tenants](https://octopus.com/docs/tenants).

Where an application contains several components, it is better to keep them within a single space; otherwise, it becomes difficult to track the currently deployed state of the application as a whole.

## Conclusion

Spaces are a valuable tool for organizing and securing the deployment-related resources you manage with Octopus Deploy. They provide smaller views over deployments and resources, but you should give due consideration to their design.
