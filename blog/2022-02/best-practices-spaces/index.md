---
title: Best practices for Spaces
description: Find out how to use Spaces effectively.
author: steve.fenton@octopus.com
visibility: private
published: 9999-01-01
metaImage: 
bannerImage: 
tags:
 - Product
 - Spaces
---

Spaces help you organize and secure your projects, environments, and infrastructure and control which team members can access them.

In this post, you'll learn how to use Spaces effectively to organize your deployments.

## An overview of Spaces

Spaces are partitions that create hard walls in your Octopus server. Deployment resources assigned to a space cannot be seen or used from another space. A *space manager* is responsible for managing each space.

![Spaces](spaces-temporary.png)

The following items are scoped to a space:

- Environments
- Lifecycles
- Projects
- Variable sets
- Deployment targets
- Tenants
- Events
- Teams
- Tasks

The [administration guide for Spaces](https://octopus.com/docs/administration/spaces) has instructions for managing Spaces.

## Benefits of Spaces

You can use Spaces to organize your deployment resources and limit who can access them.

For example, your Octopus Deploy Dashboard increases vertically as you add projects and horizontally each time you add an environment. You will also find lists getting longer as you add variable sets and deployment targets. Spaces allow you to contain the growth in each of these areas.

Many organizations need to restrict who can access particular sets of deployment resources. It is also helpful in larger organizations to make things easier for team members by limiting how much they see.

If you have to scroll through a long list of projects, Spaces are an opportunity to make things much more straightforward.

## How to design your Spaces

It may seem like a good idea to create a space for each team in your organization, but this is not the best design to follow. Because the relationship between a team and a project is likely to change, you will need to move projects between Spaces to keep them in sync with your team design. Instead, use one of the following dimensions to design your Spaces:

- Application suites
- Application audiences
- Company divisions

More information on each of these options is below.

### Application suites

Application suites are an ideal way to organise Spaces because your application design has similar considerations. For example, you might group the components of a content management system (CMS) within one space, and the components of a billing system into a second space.

### Application visibility

A less granular approach is to have separate spaces for internal applications and public-facing applications.

### Company divisions

If the company is split into sections that each develop their own applications, this is likely to provide a natural arrangement to use for Spaces. For example, if the company has divisions that offer software to different industries, each division could have a separate space.

## Useful indicators

The ideal scenario for a space is that the applications with a space are independent, deployed to dedicated targets, and have an autonomous team responsible for them. While you might not find yourself in this perfect example, it provides a useful guide when you are deciding how to design your Spaces.

If you deploy multiple applications to the same deployment targets, you should keep the deployments within the same space. 

It is possible to set up the same deployment target in more than one space using a listening tentacle. However, this complicates the permissions, and team members won't see all the deployments targeting the shared infrastructure. If there are reasons to split the deployments into multiple spaces, those same reasons are likely to mean you should have different deployment targets, too.

## Conclusion

Spaces are a key tool for organising and securing the deployment related resources you manage with Octopus Deploy. As an organisation grows, Spaces become an essential tool that helps you tackle the complexity in your deployment configuration.