---
title: Octopus Deploy 4.0 - Infrastructure Redesign
description: The environments are has been revamped into a new Infrastructure area for our upcoming Octopus 4.0 release.
author: mark.siedle@octopus.com
visibility: private
tags:
 - New Releases
 - Environments
 - Deployment targets
 - Infrastructure
 - Octopus at scale
---

For those who’ve used Octopus with more than a few hundred deployment targets, you’ll know the frustrations of the current environments screen all too well :)

Some customers use Octopus with thousands of deployment targets, some with tens of thousands of deployments targets. The rendering time to present all those machines is not great, it’s not even good ... it’s table-flipping material (and often the page will time-out during rendering once it reaches a certain memory limit). Add to that an inability to search, filter or page through results, and it simply does not scale.

There are also customers who only have the need for a handful of deployment targets, who do not want to be crippled by a solution that caters _only_ for customers operating at scale.

So we had a high-level checklist of things we wanted to improve, namely:

- Improving the rendering time for large numbers of environments and deployment targets
- Adding the ability to search and filter by various criteria
- Switching to a list layout, to make things easier to find at a glance
- Adding an overview page that provides a useful summary of your infrastructure, so you can get to what you want, fast

We were also aware of the distinction between wanting to manage environments (and deployment targets that may exist within multiple environments) and just wanting to manage deployment targets. The point is, different customers have differing needs, and trying to condense all requirements into one screen was just not feasible, so we decided to give our users options, and do both :)

## Understanding the problem

To get into this mindset, we wrote a script and loaded up 2000+ deployment targets with randomised data and opened up the (current) web-portal...

[insert screenshot here]

Yes, that’s a long-and-wrapped version of what the environments page was rendering (each of those little blue dots is a machine icon). To put this into perspective, my “full-screen screenshot” plugin ran out of memory and crashed when trying to capture all of them on my retina iMac, so I had to capture that from a VM with lesser dpi settings :)

Obviously, the first thing we noticed was a significant amount of time being spent on rendering. No paging is implemented on this screen, so if you have 2000 deployment targets, that’s 2000 HTML line-items that your web-browser needs to generate and render (not to mention all the grouping of environments and health statuses that also needs to happen).

The second thing we noticed is how difficult it is to find anything. You’re left with having to Ctrl-F in your browser, because the grid-layout makes it very difficult to scan and find things quickly.

What’s worse is, if you clicked through to a machine then clicked back, you had to wait for everything to load all over again (the rendering, the Ctrl-F to find your place again … you just wanted to scream or cry, often both). After some time of living in this state, you learned to `right-click > open deployment target in new tab` to avoid having to reload the environments page…

[insert roll over meme]
- The browser can't re-render environments
- if I never navigate away from environments

But this was actually a really good start, because we were experiencing the pain first-hand. Now we had a problem to solve and the motivation to solve it :)

## The 4.0 Solution

### Introducing the new Infrastructure area

Previously, all environments and deployment targets were located in an area called “Environments”. This environments area also had links to machine policies, proxies and accounts in the top-menu, which always felt a bit tacked on and not really related to environments at all. So firstly we decided to clean house and create a brand new area called Infrastructure, where all things infrastructure would now live (and don’t worry if you have any environment-related bookmarks, we’ve added redirects from the old v3 routes).

[insert screenshot of Infrastructure overview, showing the top-level menu)
