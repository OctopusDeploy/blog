---
title: Insights from our Optimizing Octopus webinar
description: Learn how to keep Octopus running smoothly, and easily see what’s happening, all with a little planning, maintenance and smart use of Octopus features.
author: andrew.corrigan@octopus.com
visibility: public
published: 2021-08-18-1400
metaImage: blogimage-insights-from-our-optimizing-octopus-webinar-2021.png
bannerImage: blogimage-insights-from-our-optimizing-octopus-webinar-2021.png
bannerImageAlt: clipboard and pen in front of desktop monitor with octopus logo and settings icons on screen and green tick icon
isFeatured: false
tags:
  - DevOps
---

In our recent webinar, Clear Measure’s Chris Thomas joined Derek Campbell to talk about ways to optimize Octopus Deploy for the best possible experience.

With a little planning, maintenance and smart use of its features, you can keep Octopus running smoothly and make it easier to see what’s happening.

Watch the webinar below or keep scrolling for the webinar’s key insights.

<iframe width="560" height="315" src="https://www.youtube.com/embed/M5MbNkGkIPo" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Database maintenance

As Chris notes in the webinar, it’s easy to forget Octopus Server runs on a SQL database. Some basic housekeeping with tools like **Microsoft SQL Server Management Studio (SSMS)** can boost speed for those battling a slow instance.

If you’re on [Octopus Cloud](https://octopus.com/docs/octopus-cloud), database maintenance is something you never need to worry about. Skip ahead for how to best [organize Octopus](#organize-octopus) instead.

### Check SQL reports for problems

First, we recommend running an ‘Index Physical Statistics’ report in SSMS to look for problems. 

To run the report, right-click your database from the list and select **{{Reports > Standard Reports > Index Physical Statistics}}**.

After the report has opened, check the **Operation Recommended** column for suggestions.

You can also expand items in the **# Partitions** column to see fragmentation information. 

- If fragmentation is over 35% you should schedule some time to defragment the database. 
- Anything under 30% is usually okay, but there’s room for improvement as the percentage increases.

### Set a maintenance plan

You can use SSMS's **SQL Server Maintenance Plan Wizard** to optimize and maintain your SQL database.

Expand the **Management** folder in the left pane, right-click **Maintenance Plan** and select **Maintenance Plan Wizard**.

Set the plan to run on a schedule that suits your business and select the following maintenance tasks:

- Check Database Integrity
- Reorganize Index
- Rebuild Index
- Update Statistics

Use others if needed, though we advise against shrinking the database for performance reasons.

As you progress through the wizard, you may want to change settings for each chosen task. We find the defaults are enough, but you can change things as you need. 

Run the maintenance plan once you’re happy.

If you prefer using scripts, try [Ola Hallengren’s SQL Server maintenance solution](https://ola.hallengren.com/), which achieves everything we outlined above.

## Organize Octopus

Keeping Octopus well organized not only helps performance, but also makes it easier for teams to read and navigate the dashboard. Here are a few suggestions to keep your dashboard lean and easy to navigate.

### Don’t use more environments than you need

While you can create many environments, your environment number should mirror your development pipeline (Dev, QA, Staging, UAT and Production, for example).

We advise using fewer than 10 environments to reduce:

- Performance problems
- Project redundancy and overlap
- Unnecessary clutter on the dashboard

![Octopus dashboard with ideal environment setup, showing development pipeline that includes Dev, QA, Stage, UAT and Production](environments.png "width=500")

We find some customers use more environments than needed to manage lots of projects or deployment targets. Octopus includes better options, such as Project groups and Tenants.

### Group your projects

If you have many Projects, you can group them to help reduce visual noise on your dashboard.

![Octopus dashboard showing projects in Octopus, all grouped logically using Project Groups](projects.png "width=500")

To create a project group:

1.	Click **Projects** in Octopus’s top menu.
1. Click **ADD GROUP** in the top right.
1. Give your group a name and description, then click **SAVE**.

To add an existing project to a group:

1.	Click **Projects** in Octopus’s top menu.
1.	Click on the project you need to move to the new group.
1.	Click **Settings** at the bottom of the left menu.
1.	Click **Project Group**, then select the destination group from the **Project group** dropdown menu.

To add a new project to a group:

1.	Click **Projects** in Octopus’s top menu.
1.	Click **ADD PROJECT** in the top right.
1.	Give your new project a name and click **SHOW ADVANCED**.
1.	Select a group from the **Project group** dropdown menu.
1.	Click **SAVE** to finish creating your new project.

### Use Tenants

We designed Tenants to help customers who deliver Software as a Service (SaaS), but they’re also useful to organize things like:

-	Geographical regions or data centers
-	Developers, testers or teams
-	Feature branches

![A set of tenants in Octopus, showing how they can group and manage deployment targets and demographics](tenants.png "width=500")

Check out our [documentation on Tenants](https://octopus.com/docs/tenants) for more information. 

Also, watch our follow-up webinar with Adam Close and Mark Harrison, [Better multi-tenancy deployments using Octopus Deploy](https://octopus.com/events/better-multi-tenancy-deployments-using-octopus-deploy).

### Delete your old releases and builds

It’s great practice to delete anything you no longer need from Octopus. Octopus Server is set to keep everything by default (Octopus Cloud defaults to 30 days), but you can use Lifecycle retention policies to auto-prune old releases and builds.

To check your retention policies:

1. Click **Library** in Octopus’s top menu.
1. Click **Lifecycles** from the left menu.

Here you can change retention policies for existing and default Lifecycles, or create new ones.

By setting retention policies, Octopus will delete the files after the number of days or releases you set, saving you valuable disk space.

![Screenshot showing a lifecycle's retention policy in Octopus](retention.png "width=500")

See our documentation for more detail on:

- [Lifecycles](https://octopus.com/docs/releases/lifecycles)
- [Retention policies](https://octopus.com/docs/administration/retention-policies)

## What next?

For more help, you can look through the [Octopus Deploy documentation](https://octopus.com/docs) or you're welcome to reach out via our [support channels](https://octopus.com/support).

Happy deployments!
