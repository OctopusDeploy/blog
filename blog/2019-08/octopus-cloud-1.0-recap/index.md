---
title: Reflecting on the launch of Octopus Cloud 1.0
description: We reflect on the launch of our Octopus Cloud - the SaaS version of Octopus - and how it's shaped our plans for Octopus Cloud 2.0.
author: andrew.katsivas@octopus.com
visibility: public
published: 2019-09-03
metaImage: 
bannerImage: 
tags:
 - Octopus Cloud
 - Behind the Scenes
---

![Octopus Cloud 1.0 illustration](todo.png)

## Octopus Cloud 1.0 - Reflection

About a year ago we launched Octopus Cloud (link to launch post) - a SaaS (term definition link needed) version of Octopus - as an experiment to see whether it would deliver significant value to our users, and simplify their deployment story. We wanted to empower developers to just focus on their deployment needs, and leave the management of the infrastructure to us.
Overall, we think it’s been a huge success - enough for us to invest the last year in rebuilding almost the entire platform from the ground up (link to a 2.0 launch post if there is one?). I was a part of the team that shipped the current version of Octopus Cloud, and wanted to take some time to celebrate a few of our wins, and reflect on the lessons we learnt that have shaped our redesign.

## We went down the rabbithole

From the earliest conversations about a “hosted version of Octopus” we knew we wanted Octopus itself to be a part of the deployment and orchestration story. The idea of eating your own dogfood (definition link) isn’t new in software, but it’s a great way to put yourself in the shoes of your users, and get a direct understanding of their day-to-day frustrations. This caused more than one tense internal conversation where our frustrated customers were also our teammates, but it led to quite a few of our product enhancements over the last 12 months.
We use Octopus as the tool that spins up new infrastructure and makes changes to instances out in the wild - installing Octopus onto customer’s VM as part of the process. We call this Octopus instance “The Hub”. When you consider that we also run a third internal Octopus instance that deploys The Hub (which, in turn, is deploying customer instances) it’s fair to say that things got...a little weird. 
A year on it’s still working seamlessly that way, which is a testament to how flexible and resilient Octopus Server can be - and it freed us up to focus on building the pieces of the Octopus 
Cloud story that we didn’t already have, like VMs, databases and everything in-between.

## Build it and they will come

This is one of the great fallacies, and it’s enduring because it’s just so logical - you should just build great stuff, and naturally you will have customers. Going into this experiment we knew there was interest in a cloud solution, but we didn’t know a lot of things: 
How many customers will actually use it?
What is is all going to cost?
What should we charge for it, to cover the infrastructure costs?
How many resources do we give each instance, to ensure users are successful?
Our solution was to build an MVP (link or tooltip) based on our best estimates, and test the market that way. We were so committed to this we were prepared to lose $1M on the whole project if it totally bombed and no-one used it.
Ultimately we were victims of our own success (cheeky humblebrag) - we built it, and it felt like everyone came at once! In the first <X> days we had over <Y> new cloud trials spin up - and more than a few sleepless nights for the team as we had to rapidly work around unexpected hidden service limits. This caused a few issues as we scaled, and at one point we had to pause new signups while we tried to provision more headroom.
There’s more to SaaS than autoscaling
I remember reading a tweet back in the early days of Octopus Cloud that implied that running a SaaS platform at scale was as easy as dragging the autoscale slider all the way to the right. For some apps this could very well be true, but scaling Octopus Cloud was no mean feat. To ensure that there was no way for one user’s data to mingle with another, each “cloud instance” had their own dedicated VM and database, and a large number of security configurations to prevent any funny business.

## Cloud stuff can be really expensive

We optimized Octopus Cloud for user experience and security, not for cost. We didn’t have any data on prospective revenue, so we aimed for robust architecture and figured it would all work out from there. In those early days we were spending over $100 per month to keep a single Octopus Cloud instance online, but most of our early adopters were only spending $10 per month.
Unfortunately this was one of the most painful lessons we learnt, because the deficit between what we were charging and spending was magnified by the sheer number of people using Octopus Cloud. Continued growth would mean we further amplify the problem.
Two months in we started to have some very serious conversations about our $100k USD monthly AWS spend, with very little revenue to compare it with:
AWS bill screenshot
Octopus Deploy as a company is not publically listed or VC funded (link) - we have been bootstrapped (link) and profitable since Octopus v1. However, this means that our business needs to always be sustainable, as we can’t rely on multi-million-dollar investments to bail us out. We want to continue to produce quality software for many years to come, so we needed to either make Octopus Cloud sustainable, or decide to abandon the experiment.
Ultimately, we took these lessons we were learning and started a huge body of work we internally call “Hosted v2” - a reimagining of Octopus Cloud, built to scale sustainably.

## The king is dead, long live the king!

Deciding to rebuild something from scratch is brutal - it can feel like your earlier attempt was wasted time and effort, and that work is being thrown in the bin. However, we need to recognise that a second step is only ever possible because a first step was taken, and every lesson learnt along the way is an essential input. 
A lot of things under the hood are changing with v2, but one major aspect is still the same: we remain fully committed to using Octopus as the tool to orchestrate our cloud platform. The benefit we get from being our “own customers” in this regard is immeasurable, and will hopefully drive numerous enhancements to come.
We’re also backing ourselves to learn and use a lot of technologies at the leading edge of hosting and orchestration - Terraform (link), containers (link to Azure container thingy), Kubernetes (link) and  Azure Functions (link) are just a few of these spaces we are currently working in. This approach brings its own challenges, but it will also feed into the next generation of Octopus tooling we can build with the expertise we acquire along the way.
Building Octopus Cloud v2 has been very different from v1 - where before we were guessing and flying blind, this time we have real data we can analyse to answer some of the questions we have: we know what users will spend; we know what things will cost; we know what sort of resource consumption to expect; and between those things we know how to make a platform that will truly scale.
