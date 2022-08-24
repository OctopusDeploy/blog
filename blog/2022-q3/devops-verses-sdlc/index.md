---
title: DevOps verses SDLC
description: Find out if the software development life cycle still has a place in the DevOps era.
author: steve.fenton@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: software-development-lifecycle.png
bannerImage: software-development-lifecycle.png
bannerImageAlt: Software packages moving along a conveyor belt and becoming versionsed releases.
isFeatured: false
tags: 
  - DevOps
---

The software development life cycle (SDLC), also known as the systems development life cycle, was a structured and methodical approach created in the 1960s for the development of large-scale business systems. After reaching a peak in the 1980s, traditional approaches based on the SDLC have increasingly been replaced with new approaches.

In our series on [the history of software delivery](https://www.octopus.com/devops/history/), we discussed several technical reasons that led to phased software models. Just like scientists are limited by the equipment available for their experiments, early software delivery was limited by scarce machines that were expensive to run, long compile times, and basic tools for code editing. The Web wasn't invented until 1989 and Stack Overflow didn't arrive until 2008.

## What is the SDLC?

The SDLC is a series of phases and control steps arranged in sequence, from initial need to a working product. The structure of the life cycle helped organizations ensure that a development was feasible, requirements were clear, and the correct system was delivered.

Before a formal SDLC was defined, systems would be created using an ad-hoc code and fix approach, which lacked any process or controls. Given the difficulty of development software at the time, a highly structured approach was useful to reduce the chaos.

The original SDLC is likely to be the Lincoln Labs phased model, which has 9 phases that were relevant to the technology available in the 1950s.

1. Operational plan
2. Machine and operational specifications
3. Program specifications
4. Coding specifications
5. Coding
6. Parameter testing
7. Assembly testing
8. Shakedown
9. System evaluation

Many variations of the SDLC were created with different phases, which changed as technology developed. Computers became cheap enough for every developer to have their own. The code editing tools and compilers were bundled into integrated development environments, and it eventually became possible to compile and test an application in a few minutes.

## Why the SDLC became a problem

When your primary tool for solving problems delivering software is a set of phases, you tend to solve most problems by adding more phases, or by adding further controls between them. As your process grows in size, it increases the transaction cost of each software version.

If your process represents the highest cost to software delivery, it skews your economics. With $20,000 in process overheads, you can't justify a software version with less than $100,000 in value. This led to work being combined into large batches, which created many of the problems the software industry started to face at the end of the last century.

When you create software, you need feedback from users to validate your features. Large batches delay this feedback, which drastically increases your risk.

What the SDLC taught us was that there was such as thing as *too much process*.

## Does DevOps have an SDLC

In terms of a diagram representing tasks that need to be done to deliver software, there is still a life cycle for software development. However, the formal structured SDLC of phased software delivery is no longer seen as a good practice.

If you were using an SDLC, you would arrange 20 people into 5 specialist teams to work on analysis, design, development, testing, and operations. These *horizontal teams* would perform their specialist task and work would pass from team to team, like the baton-passing in a relay race.

In DevOps you would arrange the same people into 4 cross-functional teams who could deliver software without any hand-offs. Your *vertical teams* could each deliver and run an isolated component, like the line of players moving the ball towards the try line in a game of rugby.

:::hint
In your organiztion, you are likely to mix different types of team design. Matthew Skelton and Manuel Pais created *Team Topologies* to describe different teams and interaction modes. You can use these to design healthy communication structures in your organization.
:::

Within DevOps and Continuous Delivery, there are still a series of tasks that need to be completed to deliver software. Reducing your batch size, creating autonomous vertical teams, and automating your deployment pipeline means there is no need for a formal standardized SDLC.

## A DevOps process

To design a native DevOps process, you should combine Continuous Delivery with techniques that help you discover what to build and communicate the need with the team. For example:

1. Lean Startup
2. Impact Mapping
3. Specification by Example
4. Continuous Delivery

Continuous Delivery doesn't depend on any specific technique for generating ideas. For example, if Lean Startup isn't appropriate to your organization or the software you are building, you can swap this for another technique in your product management toolkit.

## The legacy of the SDLC

For some organizations, the lasting part of the SDLC is a heavyweight set of phases and controls that limit software delivery and create high levels of market risk.

The real legacy of the SDLC should be what we learned from the first 4 decades of software delivery:

- You should work in small batches
- You should deploy frequently
- Start with a small working prototype, then evolve it
- It is possible to have too much process

With the right culture and capabilities, and a deployment pipeline that is as automated as possible, you should be able to deliver frequent high-quality software versions.

Happy deployments! 
