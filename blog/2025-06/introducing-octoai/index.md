---
title: Introducing the Octopus AI integration
description: Learn how GenAI can empower DevOps teams to deliver software faster and more reliably.
author: matthew.casperson@octopus.com
visibility: private
published: 2099-12-31-0000
metaImage: img-blog-datadog-octopus-integration-2025.png
bannerImage: img-blog-datadog-octopus-integration-2025.png
bannerImageAlt: Octopus and Datadog logos connected by plugs with little stars around the connection.
isFeatured: false
tags: 
  - AI
---

DevOps teams have embraced GenAI for many of their tasks, notably to assist with writing code, but also for other common tasks like pull request comments, documentation, debugging error messages, and more.

A question we have been exploring in Octopus is how GenAI can meaningfully improve the delivery of software. GenAI is incredibly powerful, but also has the unfortunate distinction of being "slapped on" to products in ways that add little value. So is there an opportunity to integrate GenAI with Octopus in a way that delivers genuine value to our users?

In this post I'll discuss how we plan to streamline and improve the software delivery process with GenAI, and how we plan to integrate it into Octopus.

## What is the problem we are trying to solve?

Are you part of a software company? It seems like an obvious question, and one that is straightforward to answer if your company sells software or technical services. But what about a bank, insurance provider, hospital, airline, school, supermarket, or government agency? Many of these organizations don't think of themselves as software companies.

The way to determine if you are a software company is to ask yourself: "What would happen if all the computers were turned off?"

If the answer is that your business would grind to a halt, then you are a software company. If you are a software company, then you need to deliver software to your employees and users.

DevOps teams are responsible for keeping software companies running. It is a complex job, and it will only ever get more complex. And the reality is this complexity is always going to be met with the demand to do more with less.

When you look at the metrics used to measure DevOps performance, such as DORA metrics, you will see that every one of them measures some aspect of a production deployment:

* Deployment Frequency is a direct measurement of production deployment frequency
* Mean Lead Time for Changes measures the time it takes to get changes into production
* Mean Time to Restore measures how quickly a change can be made in production to restore service
* Change Failure Rate measures how many production deployments introduce failures

Anything that prevents a DevOps team from delivering software to production reduces their performance. There is simply no way to be a high-performing DevOps team without delivering software to production.

This is the problem we are trying to solve: how do we remove the barriers to delivering software to production? Because this is the foundation of high-performing DevOps teams, and every software company demands high-performing DevOps teams.

## How do we think about the DevOps journey?

There are two stages in the DevOps adoption process that we want to focus on:

* How do we help new teams get up and running quickly, creating a solid foundation based on best practices?
* How do we reduce the friction for existing teams as they deliver software to production?

Thinking about the DevOps journey from these two perspectives helps us to identify the areas where GenAI can provide the most value. We thought about what a DevOps engineer would ask a knowledgeable colleague during these stages and how GenAI could provide an answer.

## How does GenAI get teams up and running quickly?

The first, and perhaps most obvious, way GenAI can help is by providing the ability to ask conceptual questions about Octopus, for example:

* What is a project?
* How is a project different from a runbook?
* How is an environment related to a lifecycle?

The documentation may answer some of these questions directly, but GenAI excels at providing direct answers to these kinds of questions. This GenAI functionality is table stakes these days, but still incredibly useful for new users.

The next way we will help teams get up and running is by providing the ability to build new projects, along with all the supporting resources, from prompts like:

* Create an Azure Web App project called "My Web App"
* Create an Azure functions project called "My Web App"
* Create an AWS Lambda project called "My Web App"
* Create a Kubernetes project called "My Web App"

These prompts hide a lot of complexity. To build a project with nothing more than the instruction that it deploys to Azure means making decisions about:

* Environments and lifecycle progression
* Feeds to consume application packages from
* Accounts to authenticate with Azure
* Steps to deploy the application
* Variables to configure the application
* Targets to represent the Azure Web App
* Supporting runbooks to manage the application

It is asking a lot of new users to understand all of these concepts and how they relate to each other to get even a simple project up and running.

With GenAI, we can train an LLM on hand-built template projects that bake in best practices and recreate those projects from a simple prompt. This removes the dread of the blank page and allows teams to see what a year-5 deployment project looks like, and then customize it to suit their needs.

The real power of GenAI comes from its ability to model the relationships between plain text descriptions of features and how they are implemented in Octopus. This means projects created via GenAI can be customized by adding new instructions to the prompt, such as:

* Create an Azure Web App project called "My Web App". Enable variable debugging. Enable guided failure mode. Place the project in the "Web Apps" project group. Add tenant tags based on geographic regions. Add 5 tenants named after major capital cities in Europe, the US, and Asia. Apply the tenant tags to the tenants. Link the tenants to the project.

The result of this prompt is a project based on a hand-crafted example of best practices with additional tenants and custom settings. It would take even an experienced user hours to create a project like this, but GenAI can do it in minutes. This is the power of GenAI: it puts a DevOps expert in the seat next to you.

## How does GenAI reduce friction for existing teams?

