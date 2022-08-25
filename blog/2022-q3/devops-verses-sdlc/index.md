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

The software development life cycle (SDLC) is a structured and methodical approach to software delivery. It was created in the 1960s to help with the development of large-scale business systems. After reaching a peak in the 1980s, traditional approaches based on the SDLC have increasingly been replaced with new approaches.

There were good reasons for the introduction of the SDLC, but also some good reasons to move to modern software delivery methods now the context and constraints of software development have changed.

:::hint
The SDLC is also referred to as the [*systems* development life cycle](https://en.wikipedia.org/wiki/Systems_development_life_cycle), or the *application* development life cycle.
:::

## What is the SDLC?

The SDLC is a series of phases and control steps arranged in sequence, from initial need to a working product. The structure of the life cycle helped organizations ensure that a development was feasible, requirements were clear, and the correct system was delivered.

You need to travel back in time to understand the motivation for the SDLC. Our [history of software delivery](https://www.octopus.com/devops/history/) follows the evolution of development process since 1950. Technology has played a major part in the changes. Just as scientists are limited by the equipment available for their experiments, early developers were limited by scarce machines that were expensive to run, long compile times, and limited tools for code editing.

:::hint
As an early programmer, you didn't have an integrated development environment with syntax highlighting, code navigation, or compiler warnings. People in other roles would also have no software tools to assist their work or to improve communication between roles.

Crucially, you wouldn't be able to search for answers on The Web. You'd have to work it out for yourself with the help of the manual.

- 1989 - Tim Berners-Lee invents the World Wide Web
- 1997 - Google is launched
- 2008 - Stack Overflow arrives
:::

Before the SDLC was introduced, systems would be created using an ad-hoc *code and fix* approach. With no defined process or controls, and with many technological limitations, the phased model solved many problems for organizations creating large applications.

So, the SDLC solved two problems:

1. The problems scaling the code and fix approach to large-scale systems
2. The specific technical limitations of the time

The original phased model was created at MITs Lincoln Laboratory. The model has 9 phases that were intended to directly solve the problems faced by software teams in the 1950s. Their approach allowed them to scale their development efforts, share information about the system, and document what went wrong so the knowledge could be shared with existing and future contributors.

The phases used at Lincoln Labs were:

1. Operational plan
2. Machine and operational specifications
3. Program specifications
4. Coding specifications
5. Coding
6. Parameter testing
7. Assembly testing
8. Shakedown
9. System evaluation

Many variations of the SDLC were created with different phases, which changed as both business and technology developed.

## Why the SDLC became a problem

When your primary tool for solving problems delivering software is a set of phases and control steps, you tend to solve most problems by adding more of them. As your process grows in size, it increases the transaction cost of each software version. Heavyweight processes were increasing in cost while machines were getting cheaper and compilation faster.

What the SDLC taught us was that there *was* such as thing as *too much process*.

The SDLC was introduced to solve two problems:

1. The problems scaling the code and fix approach to large-scale systems
2. The specific technical limitations of the time

While the first problem remained, the technical limitations in 1990 were nothing like those in 1960. With  technical constraints fading, the SDLC itself became the limiting factor in software delivery. The SDLC was an even greater problem in organizations that treated the process as the goal, rather than a way to achieve real organizational outcomes.

There are complex relationships between batch size, deployment frequency, and risk. No matter how rigorously you test the functional and quality attributes of the system, the market risks remain until you release the software version. You only know that a feature is useful once people are using it.

Large batches also cause a common mistake in automation economics. Common wisdom says you should automate the tasks you do most often or that you calculate the manual effort of the task multiplied by its frequency. This creates a paradox as the reason you don't perform a task frequently is that it's manual and expensive.

As well as the ability to perform automated tasks more often, you can also reduce the cost of delay and the impact of mistakes. You don't need to trade quality for speed, frequent deployments predict higher levels of quality.

In addition to the shift in technological constraints, a new competitive landscape has emerged where organizations that are slow to market have their business eaten by smaller and faster companies.

The SDLC was the correct solution at the time, but things have changed so much it is no longer a valid approach to software delivery.

## Does DevOps have an SDLC?

In terms of a diagram representing tasks that need to be done to deliver software, there is still a life cycle for software development. However, the structured SDLC of phased software delivery is no longer seen as a good practice. The constraints of the time required problems to be solved with a process and those constraints no longer exist.

Using an SDLC, you would arrange 20 people into 5 specialist teams to work on analysis, design, development, testing, and operations. These *horizontal teams* would perform their specialist task with work passed from team to team, like the baton in a relay race.

In DevOps you would arrange the same people into 4 cross-functional teams who could deliver software without any hand-offs. Your *vertical teams* could each deliver and run an isolated component, like the line of players moving the ball towards the scoring line in a game of rugby.

:::hint
In your organization, you are likely to mix different types of team design. Matthew Skelton and Manuel Pais created *Team Topologies* to describe different teams and interaction modes. You can use these to design healthy communication structures in your organization.
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
