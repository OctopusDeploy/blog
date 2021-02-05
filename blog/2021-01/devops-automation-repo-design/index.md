---
title: How to structure your Git repository for DevOps automation
description: Learn how to structure your Git repositories to store your scripts, infrastructure as code configuration files, application configuration files, docs and more. 
author: josh@duffney.io 
visibility: public
published: 2021-01-27
metaImage: blogimage-how-to-start-and-refactor-automation-repositories-2021.png
bannerImage: blogimage-how-to-start-and-refactor-automation-repositories-2021.png
tags:
 - DevOps
 - Automation
 - Git
---

In the aftermath of "automate all the things!", there are Git repositories scattered everywhere. Some repos are extremely bloated, holding one-liners, scripts, functions, modules, and configurations. While many others are stale with broken CI/CD pipelines that have gone untouched for months or years. Yet, more exist. This problem results from a failure to create and enforce a naming convention to clean up everything.

Repository design is often an afterthought, but for good reason. It's difficult to know how to structure something when you don't know what your building. It doesn't have to stay a mess, nor do you have to frontload years worth of future work into a decision that can't make today.

As a contributing member of many hot mess code bases and ring leader of "let's over engineer this", I'll share with you what's worked best for the teams I've been a member of when starting down the automation road and how to refactor a centralized repository.

## Start Simple and Centralize

![Simple repository design](simple-repo-structure.png)

Get started with one repository to rule them all! Creating a single central repository is the best way to start, which is roughly equivalent to a monolithic application. Everything is tightly coupled. Ad-hoc scripts, orchestrated automation, and infrastructure as code documents all live in the same repository. Name the repository after your team's name and let it become the source of truth for all the code developed by your team.

Keep the repository layout simple. Create general purpose directories to separate the different types of automation within the repository. For example, create a scripts directory for ad-hoc scripts, another for Terraform configuration files and so on. Add additional directories for automation that defines a process like patching. At this stage don't put too much thought into the structure and naming of directories. Your primary goal at this point is to make it as simple and easy to use as possible.

A centralized design works best when the execution of the code is still manual. Infrastructure as Code configurations, scripts, and orchestrated automation are all pulled down and run from the command line.

Comparing a centralized repository design to a monolithic app might make you think you shouldn't use it. But it's an excellent place to start if your team or organization is new to automation or infrastructure as code in general. It's something you'll grow out of eventually, but it's far better than jumping ahead to a more mature design will only add complexity, confusion, and reduce your rate of adoption.

Having everything in one place keeps things simple. It's clear how to organize files and where you should commit your code. It's easy to track changes, and there is a single CI/CD pipeline to troubleshoot.

## When is it Time to Refactor?

Friction is the best way to identify constraints. Over time your centralized repository’s codebase will start to get burdensome. And you’ll want to automate the execution of tasks and stop running everything by hand. That’s when a centralized repository starts to make things complicated.

Over time the codebase starts to get heavy. It’s not the codebase that’s the problem, it’s the workflows. The most significant disadvantage of a centralized repository is the number of workflows it contains. There is a lot of [toil](https://cloud.google.com/blog/products/management-tools/identifying-and-tracking-toil-using-sre-principles) to automate, and not all of them run at the same time.

For example, say you have a script used to generate an audit report, a PowerShell module that is used to complete a failover and Terraform code used to build the infrastructure all in a single repo. It’s not the single repo that’s the bottleneck, but the workflows. Each of these three different code bases will change and evolve at different times.

It’s at that point that the latest version of the entire code base becomes problematic. If a change to the audit script snuck in some Terraform changes bad things could happen. As a result, development slows. And that is exactly when you need to reevaluate your repository design.

In short, there are two indicators that it’s time to redesign: 
* Development has slowed.
* Risk increases by removing the manual execution of the code. 

One solution is to create a repository for every individual workflow, and while that’s an easy solution, it’s not the best solution.

## Follow the Change

<!-- TODO: Request a nice graphic from the UX team to visualise this. -->

Everything and everyone gets a repository! Is normally the first instinct when a centralized repository becomes problematic. However, that brings with it it’s own set of challenges. Instead, I’d recommend that you follow the change.

Automation is written to make a change, report a change, or to test a change. When it comes time to refactor your repository start by mapping those changes through the system. Using that data will inform your decision.

[Value stream mapping](https://www.atlassian.com/continuous-delivery/principles/value-stream-mapping) is a technique used to analyze the flow of information, people, and materials required to bring a result to a customer. Normally in the form of a product or service. It is a concept that originated in Lean manufacturing methodologies but is also prevalent in DevOps literature.

This analysis can be as complicated as you make it. Keep it simple and stick to identifying the steps in the process rather than the waste and efficiencies. Your goal is to visualize the current process instead of optimizing it.

For each different process in your central repository, go through a workflow exercise. For example, say there is an operating system patching process you’ve automated. The code lives within the central repository and is run on a monthly basis. You also have your infrastructure as code configuration within the same repository, but that is run on-demand without a schedule. Each of those processes has a different workflow and would be good candidates for separating them into their own repositories.

Creating more repositories is helpful, but it also creates more complexity. There are tradeoffs, and you need to strike the right balance. Moving code to another repository for the sake of organizing it isn’t going to motivate anyone. However, if moving code means a task can be fully automated and someone doesn’t have to wake up at 3:00 am to patch servers. Then it’s overwhelmingly worth it.

Use a simple workflow tool such as [draw.io](https://draw.io] to draft the workflows. Share the workflows with your team to get feedback. After you’ve mapped the process, start to move all the code required for that process to a different repository. Again, only decouple the code if it adds value.

## Conclusion

Starting simple is the best way to begin your automation journey. Avoid overengineering by using a centralized repository that holds all your automation processes. Refactor as soon as development slows down and friction increases. Use value stream mapping to identify the workflows that exist within your central repository. Decouple the code with the highest return on investment (ROI) to your team and add the most value