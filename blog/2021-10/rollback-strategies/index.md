---
title: Rollback Strategies with Octopus Deploy
description: Learn about the rollback options available in Octopus Deploy
author: bob.walker@octopus.com
visibility: public 
published: 2099-12-31
metaImage: 
bannerImage: 
bannerImageAlt: 
isFeatured: false
tags:
 - Product
---

Rollbacks are a easy to understand, difficult to master concept that has tripped me up in the past.  The core concept is easy; code was deployed, something broke, and there is a pressing need to quickly get back to known good state.  That core concept often gets muddied by concepts such as "roll-forward", "blue-green", "staging slots".  In this post I will walk through rollback strategies and how you can apply them to your processes today.

# Rollfoward or Rollback

Let's get this out of the way, not all releases can and should be rolled back.  Often times, the best way to fix a problem is to rollfoward.  Make the fix and push it out as quickly and safely as possible.  Rolling foward is safer for numerous reasons.

- You cannot pick and choose which piece of code to rollback in a binary.  It either all rolls back or nothing rolls back.  A team on a once a month or once a quarter release schedule will have dozens if not 100s of changes.  
- Often, database and code changes are tightly coupled together.  Safely rolling back a database **without data loss** is [extremely difficult](https://octopus.com/blog/database-rollbacks-pitfalls).
- Users will notice when something is changed and then changed back.  Especially for custom business applications used all day by the same user base.  
- With the proliferation of [Service Oriented Architecture](https://en.wikipedia.org/wiki/Service-oriented_architecture)(SOA) and its cousin [Microservices](https://en.wikipedia.org/wiki/Microservices) code changes are rarely made in isolation.  "Proper" SOA and Microservices are loosly coupled to each other and their clients.  However, in the real-world coupling exists.  A rollback to a back-end service could have downstream impacts.

Despite that, there are several scenarios when a rollback is the right solution.  Even with legacy monolith applications with a large database can be successfully rolled back in specific circumstances.  Some of those scenarios include (but not limited to):

- Styling or markup only changes.
- Back-end code changes with no public interface or model changes.
- Zero to minimal coupling with external services or applications.
- Zero to minimal database changes (new index, changing a stored procedure for performance improvements, tweaked view including additional columns on already joined tables).
- Number of changes since last release is small.

Having a rollback strategy in place, even if it is rarely used, is a valuable tool in your CI/CD toolbox.  

## Database Changes

I don't want to spend a lot of time on rolling back database changes.  That is a complex topic, and out of scope for this article.  The rollback strategies detailed below assume one of the following is true:

- You haven't made any database changes since the last release.
- Your database changes are decoupled from your code changes.
- Your database changes are backward comptiable with the previous version of your code.

## Staging Changes

When the topic of rollbacks comes up, inevitably the conversation will turn to blue/green, red/black, or canary deployments.  Those patterns all inherit from the same core concept:

1. Deploy changes to **Production** in a "staging" location users cannot access.
1. That "staging" location is tested and verified either manually and/or automatically.
1. Assuming verification passes, traffic is redirected to that "staging" location so it becomes "live."

If the verification process fails, the "live" site remains as-is.  The "staging" location is overwritten by new releases until verification passes.  

In terms of stress, that kind of **Production** is ideal.  Deploy in the middle of the day and run a bunch of tests while everyone is around and fresh.  If something goes wrong, no worries, users won't be impacted.  The switch to make "staging" live can happen during off-hours.  And you know it works since it has already been running and tested.  

_However_, implementing a blue/green, red/black, canary, or any kind of staging pattern, involves time and effort.  It quite often means everyone involved in the application's development lifecycle, from developers, to designers, to QA, to DBAs and Web Admins changing how they have done things.  

I think it is fantastic if you want to make that effort.  However, I am also pragmatic and realize not everyone has time or resources to make such a change.  And not everyone is going to implement those patterns in a **Test** environment.  

The staging pattern implementation is outside the scope of this post.  

:::hint
I hear about a lot of big tech companies implementing some form of a staging deployment.  Prior to joining Octopus, I never worked at a place that could implement those patterns.  I wrote this post to cover "rollbacks for the rest of us."
:::

# Scenarios

When I was a developer the two common rollback scenarios were:

- A change was deployed to **Test** with a critical bug and QA and the business analysts are blocked.
- A change was deployed to **Production** with a showstopping bug.

Ironically, most customers I talk to are focused on the **Production** scenario, yet the **Test** scenario happens much more often.  If you follow Octopus' core rule of [build once, deploy everywhere](https://octopus.com/blog/build-your-binaries-once), the chances of a showstopping bug making to **Production** is rare (but not impossible).

# Example Deployment Process

I am going to use the [OctoFX Sample Application](https://github.com/OctopusSamples/OctoFX) for my example rollback process.  I chose this application because it is built on .NET 4.5.2, and has the following components:

- SQL Server Database
- Windows Service
- ASP.NET MVC Website

I picked this example as it represents a typical application I've seen (and worked on).  Your database platform, back-end service, and front-end might be using completely different technology, and that is fine.  The concepts below apply to almost every process/techology.
