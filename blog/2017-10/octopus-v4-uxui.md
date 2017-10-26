---
title: Octopus Deploy 4.0 - UX and UI
description: ?.
author: jessica.ross@octopus.com
visibility: private
metaImage: 4.0_uxui_metaimage.png
bannerImage: 4.0_uxui_blogimage.png
tags:
 - New Releases
---

![Octopus 4.0 coming soon](4.0_uxui_blogimage.png)

This post is a part of our Octopus 4.0 blog series.  Follow it on our [blog](https://octopus.com/blog) or our [twitter](https://twitter.com/octopusdeploy) feed.

---
#Octopus 4.0 UX and UI

We want Octopus to grow and become a robust industry leading deployment tool, even more than it is today. But we recognise to give our users the best experience, we needed to rewrite and redesign to make sure we using the right libraries that delivered future growth and capabilities for new features. The new UI may be very different but there are a lot of pages that haven’t had the layout or content changed so you should find most areas familiar.

The purpose of Octopus 4.0 UI was to update to a modern interface, reduce cognitive load for users, redesign to accommodate for growth and above all keep providing a great user experience.

##Modern UI
It was decided to use an existing CSS and pattern library so we could focus on building a great deployment tool experience. We really liked what Google's’ Material guidelines offered but were conscious to not lose some of the iconic Octopus styling, for example, the task details style. The Material-ui library had most of the patterns that was needed and have adapted them to be our own brand. By using an existing library we utilise pre-existing patterns that most people already use so when you do upgrade to Octopus 4.0 it’s a familiar experience.
**Dashboard before**
**Dashboard after**
![Octopus 4.0 coming soon](4.0_dashboard-after_blogimage.png)

###Reducing cognitive load
The goal for each page is to be intuitive, for a user to immediately know where they are, what to do, and how to do it. We recognised some pages were busy, with text explaining what each section was and buttons that looked the same. To reduce the amount of mental effort required we applied some simple design principles:
Hierarchy for focus and flow
Colour, size and position helped 4.0 achieve a level of hierarchy, within the layout and components.

The button component uses colour to determine primary actions, secondary actions and ternary actions which usually appear inline. Any action that users don’t use often or we don’t want accidentally clicked, is now in an overflow menu.
![Octopus 4.0 coming soon](4.0_buttons_blogimage.png)

Using a page layout hierarchy allowed us to use patterns that helped visually represent a flow from high level summary, detailed summary to detailed data view. There were some cases where this didn’t work due to purpose of the data but most sections start with a card view and go to lists or table and then to forms and detailed data.
**Card view**
![Octopus 4.0 coming soon](4.0_cardview_blogimage.png)
**List view**
![Octopus 4.0 coming soon](4.0_listview_blogimage.png)
**Summary form view**
![Octopus 4.0 coming soon](4.0_summaryview_blogimage.png)
**Detailed form view**
![Octopus 4.0 coming soon](4.0_detailedview_blogimage.png)

###Keeping useful actions in close proximity
The interface was designed to have the action buttons positioned within the content area and visible at all times with a sticky section header. No more scrolling to get to the save button!
![Octopus 4.0 coming soon](4.0_sticky-header_blogimage.png)

###Consistency
By using the design layout based on hierarchy and proximity immediately addresses the issues we had on layout consistency. But we also addressed pattern inconsistency as well by using the Material-ui pattern library we were able to make sure components were reused and not recreated.

###Summary views to provide clarity
One of the biggest changes in Octopus 4.0 UI is the expanding form sections. The expanding sections allows the interface to display a summary of the configuration of a step or project settings, without the noise of the other settings.
**Summary form view**
![Octopus 4.0 coming soon](4.0_summary-view_blogimage.png)
**Detailed form view**
![Octopus 4.0 coming soon](4.0_detailed-view_blogimage.png)

##Designing for scale
The previous Octopus UI didn’t display larges amount of data very well so Octopus 4.0 needed to make sure patterns, like the expanding panels and tables, were used to show as much data on the screen as possible. An area addressed for scale issues was the environments page and the introduction of the infrastructure overview page. An issue Octopus users face, with lots of data, is finding things. So to help we went a little crazy with filters, advanced filters and pre-defined filters. I think we have some filters left to use ;)
**Environments page before**
![Octopus 4.0 coming soon](4.0_environments-before_blogimage.png)
**Environments page after**
![Octopus 4.0 coming soon](4.0_environments-after_blogimage.png)

We understand that getting used to something new can be a challenge but we believe our decision to redesign and restructure has produced an awesome Octopus 4.0 release. One that has been based on our user feedback and business goals. <This needs another sentence, will have a think about it>
