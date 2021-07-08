---
title: Octopus as a target
description: Learn how Octopus is evolving to be the best tool for managing Octopus.
author: matthew.casperson@octopus.com
visibility: private # Do not change this! This is not a public post!
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

Octopus has evolved over the years to make complex deployments easy with best practice features such as environments and targets, unique concepts like tenants and runbooks, and useful management tools like spaces.

However, while Octopus is one of the best tools available for managing external platforms and deployments, it has become clear that there is one complex platform companies are increasingly relying on for their operations that Octopus is not well suited to managing: Octopus itself. 

Complex deployments often mean thousands of tenants, targets, projects, environments, and spaces, and the only solution today to managing these resources as a group is by directly scripting the Octopus API. [The vision statement for the deploy group](https://docs.google.com/document/d/1Se7ALUyJM6zlXSJYxG_Ay7gHAaggGn8XBfgLr_VmjW0/edit) highlights that Octopus must be democratic, allowing people from various disciplines and teams to participate in releasing and deploying software is a core value for us.

To make Octopus the best tool for managing Octopus, we're proposing a new feature set called "Octopus as a target".

## What problems are we trying to solve?

Customer solutions have identified a number of issues when running Octopus as scale including:

* [How can I add/update/remove a tenant tag or project to 1000 tenants?](https://trello.com/c/aDij9iLl/148-how-can-i-add-update-remove-a-tenant-tag-or-project-to-1000-tenants)
* [How can I add/update/remove a role or environment to 100 targets?](https://trello.com/c/7Fr0VMDo/149-how-can-i-add-update-remove-a-role-or-environment-to-100-targets)
* [How can I add 40 people to a new team when I am not using external authentication?](https://trello.com/c/R9ZYofD2/164-how-can-i-add-40-people-to-a-new-team-when-i-am-not-using-external-authentication)
* [How can I convert my 200 projects to use execution containers/new azure app service step/workers?](https://trello.com/c/iIUhKHuo/217-how-can-i-convert-my-200-projects-to-use-execution-containers-new-azure-app-service-step-workers)
* [How can I add a new notification step to 200 of my projects?](https://trello.com/c/sIq3nh9q/166-how-can-i-add-a-new-notification-step-to-200-of-my-projects)
* [How can I easily coordinate multiple project deployments?](https://trello.com/c/9IZmL1Oa/159-how-can-i-easily-coordinate-multiple-project-deployments)
* [How can I see and promote all the releases in test not currently in staging?](https://trello.com/c/4IokRDDO/162-how-can-i-see-and-promote-all-the-releases-in-test-not-currently-in-staging)

