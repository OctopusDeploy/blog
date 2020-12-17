---
title: How to Start and Refactor Automation Repositories 
description: In this blog you'll learn how to start storing your automation in Git and how to know refactor when the time comes. 
author: josh@duffney.io 
visibility: private
published: 2025-01-01
metaImage: 
bannerImage: 
tags:
 - DevOps
 - Automation
 - Git
---

In the aftermath of “Automation all the things!” there are scattered repositories everywhere. A few are extremely bloated, holding one-liners, scripts, functions, modules, and configurations. While many others are stale with broken CI/CD pipelines and have gone untouched for months or years. Yet, more exist. These are from the failed enforcement of a naming convention that was supposed to clean up everything.

Repository design was an afterthought, but for good reason. You can’t know how to structure something when you don't know what your building. But, it doesn’t have to stay a mess. Nor do you have to frontload years worth of future work into a decision that can’t be made today.

As a contributing member of many hot mess code bases and ring leader of “let’s over engineer this”, I’ll share with you what’s worked best for the teams I’ve been a member of when starting down the automation road and how to refactor a centralized repository.

## Start Simple and Centralized

<!-- TODO: Request a nice graphic from the UX team to visualise this. -->

One repository to rule them all. Having a single central repository is the best way to start. It is equivalent to a monolithic application. Everything is tightly coupled. Ad-hoc scripts, orchestrated automation, and infrastructure as code documents all live in the same repository. Name the repository after your team’s name and let it become the source of truth for all the code developed by your team.

Keep the repository layout simple. Create general purpose directories to separate the different types of automation within the repository. Such as a scripts directory for ad-hoc scripts. Add other directories for automation that defines a process like patching. At this stage don’t put too much thought into the structure and naming of directories. Your primary goal at this point is to make it as simple and easy to use as possible.

A centralized design works best when the execution of the code is still manual. Infrastructure as Code configurations, scripts, and orchestrated automation are all pulled down and run from the command line.

Comparing a centralized repository design to a monolithic app probably makes you think you shouldn’t use it. But, it’s exactly where you should start if your team or organization is new to automation and or infrastructure as code in general. It’s something you’ll grow out of over time, but jumping ahead to a more mature design will only add complexity, confusion, and reduce your rate of adoption.

Having everything in one place keeps things simple. It’s clear where code should be committed. It’s easy to track changes and there is a single CI/CD pipeline to troubleshoot.

## When is it Time to Refactor?

Friction is the best way to identify constraints. Over time your centralized repository’s codebase will start to get heavy. And you’ll want to automate the execution of tasks and stop running everything by hand. That’s when a centralized repository starts to make things complicated.

Over time the codebase starts to get heavy. But, it’s not the codebase that’s the problem, it’s the workflows. The biggest disadvantage of a centralized repository is the number of workflows it contains. Let’s be honest, there is a lot of toil to be automated. And not all of them run at the same time.

For example, say you have a script that is used to generate an audit report, a PowerShell module that is used to complete a failover and Terraform code that’s used to build the infrastructure all in a single repo. It’s not the single repo that’s the bottleneck, but the workflows. Each of these three different code bases will need changes made to them at different times.

It’s at that point that the latest version of the entire code base becomes problematic. If a change to the audit script snuck in some Terraform changes bad things could happen. As a result, development slows. And that is exactly when you need to reevaluate  your repository design.

In short, there are two indicators that it’s time to redesign: development has slowed and or risk increases by removing the manual execution of the code. One solution is to create a repository for everything, while that’s an easy solution, it’s not the best solution.

## Follow the Change

<!-- TODO: Request a nice graphic from the UX team to visualise this. -->

Everything and everyone gets a repository! Is normally the first instinct when a centralized repository becomes problematic. However, that brings with it it’s own set of challenges. Instead, I’d recommend that you follow the change.

Automation is written to make a change, report a change, or to test a change. When it comes time to refactor your repository start by mapping those changes through the system. Using that data will inform your decision.

Value stream mapping is a technique used to analyze the flow of information, people, and materials required to bring an end result to a customer. Normally in the form of a product or service. It is a concept that originated in Lean manufacturing methodologies, but is also prevalent in the DevOps literature.

It is as complicated as you make it. Keep it simple and stick to identifying the steps in the process rather than the waste and efficiencies. Your goal is to visualize the current process instead of optimizing it.

For each different process in your central repository, go through a workflow exercise. For example, say there is an operating system patching process you’ve automated. The code lives within the central repository and is run on a monthly basis. You also have your infrastructure as code configuration within the same repository, but that is run on-demand without a schedule. Each of those processes has a different workflow and would be good candidates for separating them into their own repositories.

Creating more repositories, creates more complexity. And for that reason there has to be a trade off. Moving code to another repository for the sake of organizing it isn’t going to motivate anyone. However, if moving code means a task can be fully automated and someone doesn’t have to wake up at 3:00 am to patch servers. Then it’s overwhelmingly worth it.

Use a simple workflow tool such as draw.io to draft the workflows. Share the workflows with your team to get feedback. After you’ve mapped the process, start to move all the code required for that process to a different repository. Again, only decouple the code if it adds value.

## Conclusion

Simple is the best way to begin your automation journey. Avoid overengineering by using a centralized repository that holds all your automation processes. Refactor as soon as development slows down and friction increases. Use value stream mapping to identify the workflows that exist within your central repository. Decouple the code that has the highest ROI (return on investment) to your team and adds the most value.