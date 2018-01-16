---
title: "Octopus Deploy 2018 Roadmap"
description: "This post outlines our roadmap for 2018, and all of the improvements we're planning to make to Octopus over the next year."
author: paul.stovell@octopus.com
visibility: private
metaImage: metaImage-roadmap2018.png
bannerImage: blogImage-roadmap2018.png
tags:
 - Company
---

Yesterday I posted our reflections on 2017. I wrote about some of the things we accomplished, some of the things we didn't accomplish and reasons why, and some challenges that we face. Today, I want to share our roadmap for 2018. 

![Roadmap for 2018](blogImage-roadmap2018.png)

## Themes

There are five key themes to the work we'll be doing in 2018. 

1. Bringing a cloud-hosted version of Octopus to market
2. Expanding Octopus outside of the Microsoft ecosystem
3. Evolving Octopus into a complete DevOps tool
4. Continually improving the performance, scalability, stability and user experience

## Cloud Octopus

When we first built Octopus, most of the world were still deploying applications on-premises. Octopus was a VM that you ran alongside many hundreds of other VM's in your enterprise. And when we speak to larger customers, this is actually still the norm - despite what cloud vendors would like you to believe, there's still plenty of servers on-premises. 

That said, if every app you are deploying is a Lambda function, or an Azure Website, or a Docker container, it's a drag to have to look after a VM just for Octopus. Our friend [Jeremy Cade](https://twitter.com/jcade83?lang=en) once said:

> Octopus is the most important thing I use, and it's the only VM I have left.

A cloud hosted version of Octopus is a lot of work, and it's something we're committed to delivering soon. Unlike many of the efforts on this roadmap, which tend to be projects with a defined idea of "done", this cloud hosted version of Octopus is going to be a long-term investment and a big change to us organizationally. 

We've been working on it for months now, and In January 2018 we'll be launching a closed alpha, followed by an open beta sometime before the end of March. Initially, we'll keep it as similar to the on-premises version of Octopus as possible, and as we learn more from running it in production we'll evolve and optimize it for the cloud.  

## Expanding outside of the Microsoft ecosystem

In years gone by, we had Microsoft shops and Java shops, and it was pretty clear where most companies stood. Octopus started purely as a .NET deployment tool, but we recognized that this would need to change years ago. People and companies identify less as a ".NET Developer" and more as "Developer, who happens to do .NET, along with some Node, and a few other platforms". 

We already support deployments to non-Windows platforms via SSH (and without a Mono dependency), running Bash scripts, and we made great progress in deploying Java applications in 2017. To date our success has mostly been with Microsoft shops who occasionally use non-Microsoft technology. 

This year we're going to continue this theme:

- We'll bring first class AWS support, up to par with Azure support
- We'll bring Tentacle to Linux (to allow polling connections)
- We'll add support for running Python & Ruby scripts (in addition to PowerShell, C#, Bash, etc. today)
- We'll expand our existing Docker support to work with Kubernetes

## Evolving Octopus into a complete DevOps tool

Octopus deploys applications, but it can be used to deploy so much more. When you think about it:

- Octopus knows all about your applications - what languages they're built in, what web server they run under
- Your deployment pipeline (dev, test, production, etc.)
- How your applications are configured in all of those
- The infrastructure that everything runs on

With all of this at our fingertips, we can do so much more than deploying. 

- With the help of AWS Cloud Formation, Azure Resource Group Templates or Terraform, we could provision and deploy to environments for every feature branch you create
- We could provision test environments for testers, and de-provision them after 5pm
- We can run processes other than deployment. Any kind of "maintenance" or "operations" process or run book could be modelled in Octopus:
  - Disaster recovery failover processes 
  - Health check & compliance check processes 
  - Restoring a production database backup your test environment
  - Running automated UI tests against a recently deployed environment
- Running these on a schedule, manually with parameters, or as a hook when an alert is raised by a monitoring tool

We've spent a lot of time designing these ideas and stories, and we're confident we can achieve this without compromising the "does one thing and does it well" philosophy of Octopus. 

## Scalability and continual improvements

Octopus is becoming the standard deployment tool at more and more companies, and we want to ensure it's a seamless experience. This year we'll be making big improvements for those of you who use Octopus heavily:

- A new "Spaces" feature will let you split up your Octopus server by teams or departments, giving each their own isolated projects, environments, delegated permissions, and so on. 
- "Workers" will allow scripts that are today marked "Run on Octopus" to run somewhere else (like a build agent). This will make it easier to isolate your Octopus server from side effects. 
- We'll be making big improvements to the scalability and performance of Octopus. This is necessary for ourselves with hosted Octopus, but will benefit Octopus customers greatly too. 
- We'll continue to hunt for improvements to performance, quality and user experience. 
- We'll build [Remote release promotions](https://octopus.com/blog/remote-release-promotions-rfc).
- We'll continue to address UserVoice suggestions, and we'll ensure we do something about anything that makes it into the top 10 (or, clearly decide we aren't going to do them). Today those are:
  - **[700 votes](https://octopusdeploy.uservoice.com/forums/170787/suggestions/12948603)** Composite Step Templates
   - **[570 votes](https://octopusdeploy.uservoice.com/forums/170787/suggestions/6169634)** 'Dry Run' Deployment
   - **[447 votes](https://octopusdeploy.uservoice.com/forums/170787/suggestions/6599104)** Recurring Scheduled Deployments
   - **[442 votes](https://octopusdeploy.uservoice.com/forums/170787/suggestions/15698781)** Version control configuration
   - **[365 votes](https://octopusdeploy.uservoice.com/forums/170787/suggestions/6986441)** Permission attributes for variable sets, library variable sets, or even variable
   - **[317 votes](https://octopusdeploy.uservoice.com/forums/170787/suggestions/9196032)** Output variables for offline drops
   - **[310 votes](https://octopusdeploy.uservoice.com/forums/170787/suggestions/9811932)** Allow project dependencies - so deploying one project would automatically deploy all dependent projects
   - **[288 votes](https://octopusdeploy.uservoice.com/forums/170787/suggestions/5731235)** Environment groups
   - **[263 votes](https://octopusdeploy.uservoice.com/forums/170787/suggestions/17930755)** Support for Kubernetes
   - **[257 votes](https://octopusdeploy.uservoice.com/forums/170787/suggestions/6298548)** Allow approval step for scheduled deployment happen before actual deployment

## Wrapping up

> "N battle plan ever survives first contact with the enemy"
>  - Helmuth von Moltke

We may not accomplish everything on this list in 2018, and we may do other things that aren't on this list as the year evolves. I'm sure everyone reading this will understand that that's the nature of a roadmap - this is our attempt to outline our plans and goals for the year, and it is likely to change. That said, I hope there's something on this list for everyone. Happy deployments!