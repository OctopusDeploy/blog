---
title: DevOps verses SDLC
description: Find out if the software development life cycle still has a place in the DevOps era.
author: steve.fenton@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: Man sitting on top of container with green circle with a power up icon
isFeatured: false
tags: 
  - DevOps
---

## What is the SDLC

The software development life cycle (SDLC) has been around in various forms since the dawn of software engineering. It was introduced to resolve issues with the previous way of writing software

The software development life cycle is a specific version of the systems development life cycle (which has the same acronym).

## The purpose of the SDLC

Delivery all the software, at high quality, and the lowest cost

## What has changed

"Code and Fix". When you develop software with no process, you realize at some point that some level of process helps. When adding some process helps, it can be tempting to introduce more process, in the hope of getting more benefit. Code and fix still exists in some organizations where the software remains small enough to be managed with no formal process. Code and fix stops being viable once a software system grows beyond a trivial size. You can normally tell when this happens as the test cycle (the time between features being completed, and features being released) will grow faster than the software.

"Heavyweight". The introduction of process improves performance compared to code and fix style development. A one sentence goal is a better plan than no plan, and a sketched out design is better than no design. Over time, the plan and the design become more detailed and early experiments find that a little more detail does in fact improve software delivery performance.

"Agile". When you first learn to cook, you discover that adding seasoning makes food taste better, until you add too much and the dish becomes irretrievably inedible. It is not until someone uses *too much process* that the limit of process can be discovered. This is the tipping point where process stops brining benefits. If you continue past this point, you start to negatively impact software delivery performance. This discovery was made in the 1980s, but it took a long time for the discovery to reach the mainstream. Some organizations resist the discovery even now.

The tipping point for too-much process is further left than you think... as will all curves of this type, you want to remain just below the tipping point, so you have a little room to add more process should you need it. If you operate at or above the tipping point and need to add more process, you immediately hit drops in quality or productivity. People tend to overshoot the curve, often never experiencing the peak performance, so they don't notice that the additional process is limiting their software delivery.

![Capability decreases after the process tipping point](process-capability.jpg)

This chart will differ based on your team and culture. It is hard to measure process weight as organizations with too much process often can't tell whether people are working on real work activities or artificial work activities that result from the process. Often, they designate process-related activities as *real work* and fail to spot when individuals become fully dedicated to process and delivery no value-adding work.

Process and culture... a bureaucratic culture loves a prescriptive process. By standardizing exactly how work is done, they believe it will be easier to train new people and pass important industry audits. The problem is that heavyweight process makes many activities outside of software delivery simpler, while increasing the difficulty of software delivery itself.

One of the strange themes of resistance to the discovery that there is such a thing as "too much process" is that as heavyweight methods fall out of use, they are re-invented with new names. Too much process is too much, even if you use agile-sounding words.

Technically, quite a lot hasn't changed. All of the authors of papers on SDLC over the years have agreed that you should work in small batches.

Herbert D. Benington (Lincoln Labs phased model, 1956) said that a small prototype should be created, and then evolved towards an operational product. Winston Royce (phased software development, 1970) said that risk increases in line project size and control steps add to the cost of the project. Barry Boehm (spiral model, 1988) encouraged people to work in small batches and tailor the process for each increment according to the risks.

Despite their advice, people seemed to fixate on the process-element - perhaps driven by maturity models that required organizations to demonstrate that they produced a list of specific documents throughout their projects.

Lightweight methods emerged during the 1990s that tried to reduce the fixed overheads of the software delivery life cycle. More people were realizing that communication structures were more important than process and working software had a higher value that the required documentation list. While some process and documentation was needed, it was nowhere near the amount in use at the time.


Investment in planning and design was an intuitive way to control projects, but we learned that reality runs counter to this assumption. The business risk of a change increases hourly until you release it and discover whether the software does what the users need. Anything that delays a release increases risk.

We've gained decades of experience that shows us increasing investment in activities on the left (planning and design) doesn't increase the outcomes on the right (high-quality software that users love).

Another recent shift in thinking is where to place the programming activity. Traditionally, writing code has been part of a *production* phase that occurs *after* the design phase. Increasingly, we recognize that writing code is part of the *design* phase. The *production* phase of software delivery starts when the code change is committed to version control. The production phase should be managed as a [deployment pipeline](https://octopus.com/devops/continuous-delivery/what-is-a-deployment-pipeline/) and [deployment pipelines should be automated as much as possible](https://octopus.com/devops/continuous-delivery/automate-everything/).


## The SDLC still exists

At it's most general, the SDLC will always exist. You have a series of steps that have to be done in a specific order. Unlike early SDLCs, you can have fewer steps and you can make each step really small. For example, your product manager could spot an opportunity to add amazing value to the product on Monday morning. They could share this idea with the development team, and have the first code change available within 3 hours. The prototype could be tried out and the idea released, refined, or dropped. All within a day.

There were still stages

1. An idea was generated
2. The team worked out how they might make it happen
3. A change was made to the software
4. Feedback was obtained about the change

While this is less formal and has fewer explicit controls than a traditional SDLC, it results in higher-quality software and faster feedback loops.


## NOTES



But the way we manage quality and cost has changed dramatically over the past decade as traditional project controls have been discredited.

Traditional SDLC
- Requirements
- Design
- Development
- Testing
- Integration
- Deployment
- Operation
- Maintenance

KEY CHANGES
- Design and development were traditionally seen as separate phases, but it is better to consider the coding to be part of the design. This allows you to embrace iterative and incremental progress towards solving a requirement
- Build quality in, means integration and testing are both continuous. They shouldn't be stand-alone stages, but be part of the daily work
- Deployment pipelines are now automated, with each change built, tested, and available to deploy to production, which has many benefits
- Traditional models assumed each stage would validate the previous stage, for example the design stage would highlight errors in the requirements stage. This turned out to be misleading as the requirements stage is only really validated after the software is deployed and users attempt to use it.

Traditional SDLC vs DevOps cost control

The traditional SDLC hoped to spend money up-front to reduce errors and ensure the correct software was built. Investing in detailed requirements and designs was intended to ensure money spend developing the software was all "good spending".

Traditional SDLC increased the effort on requirements to ensure they were detailed and correct. This didn't substantially alter the outcome as people only really know if software solves their problem once they get it.

DevOps controls cost by working in small batches and releasing software versions frequently. Teams are allowed to make decisions as they can respond faster than if they have to await external decisions. The deployment pipeline is automated to decrease errors, increase quality, and reduce the transaction cost of deployments

Traditional SDLC vs DevOps quality control

In the traditional SDLC, testing is a phase or a number of phases that happen once a set of changes is available. When issues were discovered, the batch would be on hold until fixes were delivered and re-tested. This cycle increased the size of the batch and required re-testing of the updated software. If testing revealed a critical misunderstanding in how the software had been designed, or highlighted conflicting requirements, there could be expensive rework required.

In DevOps, quality is a daily part of the process. Instead of waiting for a number of changes to take place, each change is tested. This is assisted with test automation, but leaves space for human ingenuity where necessary - for example UX testing. If a developer commits a change that fails an automated test, no human need spend time manually testing the version. The developer can respond almost immediately to the error and fix the software straight away.

Research confirmed that small batches and frequent deployments results in higher quality software, which means the traditional SDLC has lower quality than DevOps.



Happy deployments! 
