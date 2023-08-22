---
title: "Paved versus golden paths"
description: The terms paved paths and goldens paths are often heard in Platform Engineering circles, but they are different concepts.
author: steve.fenton@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 
isFeatured: false
tags: 
  - DevOps
---

If you are following the Platform Engineering trend, you'll have heard people talking about paved paths and golden paths. They are sometimes used as synonyms, but can also be different approaches.

## Paved paths

If you were a city planner designing a park, you'd need to provide areas for people to stop as well as routes for people to pass through. The perfect park is a public space that you can see, walk through, and use for recreational activities. One of the tricky parts of park design is where to place the paths.

People who want to take a leisurely walk would prefer a winding scenenic stroll around the park, but people passing through on their way from the coffee shop to the office prefer more direct routes. Most parks offer either winding routes through the park or a series of direct paths that form a large X criss-crossing the park.

As an alternative to planning the routes through the park, you can let people use it for a while. People using the park will wear tracks in the grass that indicate where paths may be most useful. They literally vote with their feet. Building these paths after the demand for a route is visible means you are more likley to put them in the right place, though you can't please everyone.

There are also dangers to this approach, as [Sam Walter Foss warned](https://poets.org/poem/calf-path) in his 1895 about how a playful calf ended up accidentally designing the main street in a large city, as it was build around the path it made through the woods some 300 years earlier.

## Paved paths in software

You can use the paved paths technique in software. You can observe how users currently achieve a goal, and use what you find to generate a design for the software.

Before source control systems were created, a common technique for avoiding collisions with changes made by other developers was the source wall. To create a source wall, you'd write each file name on a sticky note and put it up on a spare wall.

If you wanted to edit a file, you'd go to the source wall and fild the file you wanted to change. If the sticky note was on the wall, you could take it back to your desk and make your edit. If you couldn't find the sticky note, you had to wait for it to come back before you made your change.

This manual process meant your changes would never clash or overwrite another developer's changes.

The first source control system paved this path. You would check out a file and the system would prevent another developer changing it until you checked it back in. This pattern was the paved path equivalent of the source wall.

If you are using a modern source control system, you'll notice that it doesn't work this way. That's because the paved path has been replaced by something better. A golden path.

## Golden paths

Going back to the city park example, if you had a design in mind for the use of different spaces, you might want to tempt people to take a slightly longer route that allows you to make better use of the overall space.

Instead of optimizing the park for commuters, you want to balance the many different uses. In this case, you'd need to find ways to attract people to your preferred route to avoid them damaging the grass and planting.

In Brisbane, the park in the South Bank area features just such a path. Instead of offering an efficient straight line between common destinations, it has sweeping curves. Along it's entire length, the path has an arbour that provides shelter from the hot sun from light showers.

Instead of attempting to block other routes with fences, people are attracted to the route because they can stay cool and dry. The Brisbane grand arbour walk is 150 metres longer than a straight-line route, but it creates spaces for restaurants, a pond, a rainforest walk, and a lagoon.

Golden paths are a system-level design technique. They are informed by a deep understanding of the different purposes of the space.

## Golden paths

In Platform Engineering, golden paths are just like Brisbane's grand arbour. Instead of forcing developers to do things a certain way, interal developer platforms are designed to attract developers in a positive way by bringing benefits.

In Platform Engineering, the golden paths are a route towards alignment. Say you have five teams all using a different continuous integration tool. As a platform engineer, you'd work out the best way to build, test, and package all the software and provide this as a golden path. It needs to be better than what developers currently do and easy to adopt, as you can't force it on a team. The teams that adopt the golden path have an easy life as far as their continuous integration activities are concerned and this is how you convince the other teams to eventually opt in.


If you build a platform by encoding the current practice within your internal developer platform, all you do is shift the pain to the platform team.

The concept for your internal developer platform should start with an understanding of developer pain. Once you know the things getting in their way, you can build a platform that makes their job more satisfying. 

Instead of asking where do they currently walk, find out more about where they are going and why.


Happy deployments!
