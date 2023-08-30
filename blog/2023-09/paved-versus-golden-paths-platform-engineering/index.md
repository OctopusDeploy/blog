---
title: Paved versus golden paths in Platform Engineering
description: Find out the critical difference between paved paths and golden paths in Platform Engineering.
author: steve.fenton@octopus.com
visibility: public
published: 2023-09-04-1400
metaImage: blogimage-pavedvsgoldenpaths-2023.png
bannerImage: blogimage-pavedvsgoldenpaths-2023.png
bannerImageAlt: Curved path running between two points on a map.
isFeatured: false
tags: 
  - DevOps
  - Platform Engineering
---

If you follow the [Platform Engineering](https://octopus.com/devops/platform-engineering/) trend, you'll have heard people talking about paved paths and golden paths. They're sometimes used as synonyms but can also reflect different approaches.

In this post, I discuss the critical difference between paved paths and golden paths in Platform Engineering.

## Paved paths

If you were a city planner designing a park, you'd need to provide areas for people to stop and routes to pass through. The perfect park is a public space you can see, stroll through, and use for recreational activities. One of the tricky parts of park design is where to place the paths.

People who want to take a leisurely walk prefer a winding scenic stroll with pleasant views. But people passing through from the coffee shop to their office prefer more direct routes. Most parks offer winding trails through the park or a series of direct paths forming a giant X crisscrossing the park.

As an alternative to planning the routes through the park, you can let people use it for a while. People crossing the park will wear tracks in the grass that indicate where paths may be most helpful. They literally vote with their feet. Building these paths after the demand for a route is visible means you're more likely to put them in the right place, though you can't please everyone.

This approach is also dangerous, as [Sam Walter Foss warned](https://poets.org/poem/calf-path). His 1895 poem tells how a playful calf influences the design of a large city. The city's main road gets built around the trail the calf made through the woods some 300 years earlier.

## Paved paths in software

You can use the paved path technique in software. You can observe how users currently achieve a goal and use what you find to generate a design for the software.

Before people created software source control systems, the source wall was a common way to avoid change collisions. To create a source wall, you'd write each file name on a sticky note and add it to the wall.

If you wanted to edit a file, you'd go to the source wall and find the file you wanted to change. If the sticky note was on the wall, you could take it back to your desk and make your edit. If you couldn't find the sticky note, you had to wait for its return before making your change.

This manual process meant your changes would never clash, and you'd never overwrite another developer's changes.

The first source control system paved this path. You'd check out a file, and the system would prevent another developer from changing it until you checked it back in. This pattern was the paved path equivalent of the source wall.

If you use a modern source control system, you'll notice it doesn't work this way. That's because something better has replaced the paved path - a golden path.

## Golden paths

Going back to the city park example, if you had a design in mind for the use of different spaces, you might want to tempt people to take a slightly longer route that lets you make better use of the overall space.

Instead of optimizing the park for commuters, you want to balance the many different uses. In this case, you'd need to find ways to attract people to your preferred route to avoid them damaging the grass and planting.

In Brisbane, the park in the South Bank area features just such a path. Instead of offering an efficient straight line between common destinations, it has sweeping curves along its entire length. The path has a decorative arbor that provides shelter from the hot sun and light showers.

Instead of attempting to block other routes with fences, people are attracted to the path because they can stay cool or dry. The Brisbane Grand Arbor walk is 150 meters longer than a straight-line route, but it creates spaces for restaurants, a pond, a rainforest walk, and a lagoon.

Golden paths are a system-level design technique. They're informed by a deep understanding of the different purposes of the space.

## Golden paths in Platform Engineering

In Platform Engineering, golden paths are just like Brisbane's Grand Arbor. Instead of forcing developers to do things a certain way, you design the internal developer platform to attract developers by reducing their burden and removing pain points. It's the optimal space between *anything goes* and *forced standardization*.

Golden paths provide a route toward alignment. Say you have 5 teams, all using different Continuous Integration tools. As a platform engineer, you'd work out the best way to build, test, and package all the software and provide this as a golden path. It needs to be better than what developers currently do and easy to adopt, as you can't force it on a team. The teams that adopt the golden path have an easy life as far as their Continuous Integration activities are concerned. Nothing makes a platform more attractive than seeing happy users.

When done well, an internal development platform may *feel* like a paved path to the developers, but it should reduce the overall cognitive load. This often involves both consolidation and standardization.

You won't solve all developer pain at once. Platform engineers will need to go and see what pain exists and think about how they might design a product that will remove it. When you start this journey, it's worth understanding the [patterns and anti-patterns of Platform Engineering](https://octopus.com/devops/platform-engineering/patterns-anti-patterns/).

## Take the high road

World champion weightlifter Jerzy Gregorek once said: "Hard choices, easy life. Easy choices, hard life." You need to make many hard choices to create a great internal developer platform.

You have to decide what problems the platform will solve and which it won't. You need to determine when a feature should flex to meet the needs of a development team and when you should let them strike out on their own path.

These hard choices are the difference between a golden path and a paved path.

With a paved path, you can reduce the burden on developers; the pain just moves into your platform team. A golden path will reduce the total cognitive load for everyone by dedicating the platform team to its elimination.

Happy deployments!