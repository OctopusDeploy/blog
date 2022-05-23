---
title: Comparing Lean, Agile, and continuous delivery
description: TBC
author: steve.fenton@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: placeholderimg.png
bannerImage: placeholderimg.png
bannerImageAlt: TBC
isFeatured: false
tags: 
  - DevOps
---

After two decades, The Agile Manifesto is still a big influence on software development teams around the world. Now, with DevOps and continuous delivery gaining traction, it's a good time to compare the 5 principles of continuous delivery to the 12 Agile principles. In his book *Continuous Delivery Pipelines*, Dave Farley places "apply Lean and Agile principles" in his 7 essential techniques, so how well do they align? Let's find out.

## Agile and continuous delivery

Continuous delivery uses the deployment pipeline as a focusing lens for the continuous improvement of the software delivery process. There are specific capabilities, techniques, and principles involved and there's a strong preference for automation where it's appropriate. Research has found the technical capabilities of continuous delivery are a direct impact on the success of an organization. A continuous delivery pipeline with fully automated approval stages is sometimes called _continuous deployment_, where no human intervention is required after the code is integrated into the main or trunk branch in version control.

Agile is a set of principles attached to a collective manifesto that emerged from the lightweight and adaptive software development movement. The Agile Manifesto was written to represent a set of values to use as a litmus test for development methods. Rather than defining a model or framework, the values and principles could guide the creation and evolution of many different approaches. Some examples of popular Agile methods include Extreme Programming, Scrum, and Dynamic Systems Development Method (DSDM).

Continuous delivery was influenced by Agile and Lean, and took it's name from the first principle of The Agile Manifesto: "Our highest priority is to satisfy the customer through early and continuous delivery of valuable software". Continuous delivery seeks to deliver the maximum value for the least effort and requires software be in a constant state of readiness. It uses automation to clear the path for fast and frequent delivery of software.

### The Agile Manifesto

When the original authors wrote down the manifesto and principles for Agile, they agreed that they would let it reflect a point in time. Alistair Cockburn recalled this agreement in an [article](https://web.archive.org/web/20170626102447/http://alistair.cockburn.us/How+I+saved+Agile+and+the+rest+of+the+world) from June 2017:

 > ...it was a free-for-all, that is to say, there was no given purpose or agenda. Self-organization at it’s finest, held together by deep respect and generous listening on all sides. What emerged was what these exact 17 people could produce together at that particular moment in history. We agreed at the end never to update the manifesto for that exact reason.

Many other people have written alternate versions, but most of these represent a well considered but single-person perspective. Two exceptions to this are [Modern Agile](https://modernagile.org/) and [Heart of Agile](https://heartofagile.com/). With Modern Agile, Joshua Kerievsky adds more emphasis on human factors, while in Heart of Agile, Alistair Cockburn seeks to strip away unnecessary "decoration" from Agile and return it to "the core elements that really matter".

These three approaches will be used for a comparison of Agile and continuous delivery. 

### Different method approaches

Most processes are based around a variation of the plan, do, check, act cycle (PDCA). In software delivery, it is common to find four specific activities that need to be done in a specific order:

1. Innovation
2. Communication
3. Delivery
4. Review

The innovation stage is where you have to come up with an idea that has some value to a customer. This might be a data-driven exercise, such as customers sharing a problem they would pay someone to solve. In some cases it might just be an intuitive idea based on experience. Often, it's a combination of the two. You need some way to come up with an idea for a feature or product.

The communication stage is where you share the idea. You might need someone to agree the funding and you'll certainly need to get the idea firmly embedded into the minds of the people who are going to make it happen. In traditional projects, this was often a detailed document, in modern software teams this is often a more interactive process, such as a specification workshop or kick-off meeting.

The delivery stage is where the idea is turned into working software.

In the review stage, you check whether the software has meaningfully solved the problem. You may be done, or you may find you need to do more before you have something of value. You might decide to continue work in the same area, in which case you loop back around for a new innovation, or you might move onto something new instead.

You'll find that most modern software development methods act in one or two of these four stages. This is not a problem, as it allows you to combine different ideas to create an end-to-end process that works for your organization.

| Innovate       | Communicate            | Delivery                    | Review         |
|----------------|------------------------|-----------------------------|----------------|
| Lean Startup   | Scrum                  | Extreme Programming         | Lean Startup   |
| DIBB (Spotify) | Kick off               | Continuous Delivery         | DIBB (Spotify) |
| Impact Mapping | Specification Workshop | Behavior-Driven Development | Impact Mapping |
| Impact Mapping | Scrum                  | Continuous Delivery         | Impact Mapping |

The selections you make will be influenced by your organization's attitude towards adoption. In the 2022 Culture and Methods report from [InfoQ](https://www.infoq.com/articles/culture-trends-2022/), Agile and Scrum were firmly established for late majority adopters, while the early majority has moved onto DevOps and pragmatic agility. The early adopters have already moved into cultural spaces, addressing psychological safety, developer experience, and async working. Additional work will be required if you need to move your organization from a late majority position.

Techniques such as Kanban provide a mechanism to visualize whatever value stream you currently have, limit your batch size, and continuously improve it.



DevOps model for the relationships between practices (which places CD firmly in the picture) provides signposts to cultural and organizational capabilities that complement the practices of CD.

Let's look at the principles and then compare them...

## CD principles

https://continuousdelivery.com/principles/

1. Build quality in
1. Work in small batches
1. Computers perform repetitive tasks, people solve problems
1. Relentlessly pursue continuous improvement
1. Everyone is responsible


## The principles of The Agile Manifesto

https://agilemanifesto.org/principles.html

1. Our highest priority is to satisfy the customer through early and continuous delivery of valuable software.
1. Welcome changing requirements, even later in development. Agile processes harness change for the customer's competitive advantage.
1. Deliver working software frequently, from a couple of weeks to a couple of months, with a preference to the shorter timescale.
1. Business people and developers must work together daily throughout the project.
1. Build projects around motivated individuals. Give them the environment and support they need, and trust them to get the job done.
1. The most efficient and effective method of conveying information to and within a development team is face-to-face conversation.
1. Working software is the primary measure of progress.
1. Agile processes promote sustainable development. The sponsors, developers, and users should be able to maintain a constant pace indefinitely.
1. Continuous attention to technical excellence and good design enhances agility.
1. Simplicity--the art of maximizing the amount of work not done--is essential.
1. The best architectures, requirements, and designs emerge from self-organizing teams.
1. At regular intervals, the team reflects on how to become more effective, then tunes and adjusts its behavior accordingly.



## Mapping

|                                                   | Build quality in | Work in small batches | Computers/Humans | Continuous Improvement | Everyone is responsible |
|---------------------------------------------------|------------------|-----------------------|------------------|------------------------|-------------------------|
| Early and continuous delivery / valuable software |                  | ✅                     | ✅                |                        |                         |
| Welcome / harness change                          |                  | ✅                     |                  |                        |                         |
| Deliver frequently                                | ✅                | ✅                     | ✅                |                        |                         |
| Work together                                     |                  |                       |                  |                        |                         |
| Trust and motivation                              |                  |                       | ✅                |                        | ✅                       |
| Face-to-face conversations                        |                  |                       |                  |                        |                         |
| Measure progress / working software               |                  | ✅                     |                  |                        |                         |
| Sustainable pace                                  |                  | ✅                     |                  |                        |                         |
| Technical excellence and design                   | ✅                |                       |                  | ✅                      |                         |
| Simplicity / scope reduction                      |                  | ✅                     |                  |                        |                         |
| Self-organizing teams                             |                  |                       |                  |                        | ✅                       |
| Reflect regularly                                 | ✅                |                       |                  | ✅                      |                         |

Most of the agile principles are covered by one of the CD principles

A section on how each agile principle is covered by CD...

Early and continuous delivery / valuable software

> Our highest priority is to satisfy the customer through early and continuous delivery of valuable software.

Welcome / harness change

> Welcome changing requirements, even later in development. Agile processes harness change for the customer's competitive advantage.

Deliver frequently

> Deliver working software frequently, from a couple of weeks to a couple of months, with a preference to the shorter timescale.

Work together

> Business people and developers must work together daily throughout the project.

Trust and motivation

> Build projects around motivated individuals. Give them the environment and support they need, and trust them to get the job done.

Face-to-face conversations

> The most efficient and effective method of conveying information to and within a development team is face-to-face conversation.

Measure progress / working software

> Working software is the primary measure of progress.

Sustainable pace

> Agile processes promote sustainable development. The sponsors, developers, and users should be able to maintain a constant pace indefinitely.

Technical excellence and design

> Continuous attention to technical excellence and good design enhances agility.

Simplicity / scope reduction

> Simplicity--the art of maximizing the amount of work not done--is essential.

Self-organizing teams

> The best architectures, requirements, and designs emerge from self-organizing teams.

Reflect regularly

> At regular intervals, the team reflects on how to become more effective, then tunes and adjusts its behavior accordingly.




Technically speaking, CD principles don't explicitly cover all the agile principles of working together and face-to-face conversations. The original CD book was written by Jez Humble and Dave Farley using different text editors, in different countries, with version control and automated builds. The authors and editors all worked remotely and asynchronously with a continuous delivery pipeline in place... i.e. they followed their own recommendations.

Rather than omitting the principles of working in the same space at the same time, CD specifically challenges that assumption. People should work together continually, but not necessarily in the same location. Remote working is seen as a competitive advantage, allowing modern employers to access talent that might not be available if geography was a key factor.

Conclusion is - it's aligned but not afraid to challenge some of the assumptions. Best not to be judgmental  of concepts that may have been more relevant in 2001 than they are two decades later. You don't have to throw out The Agile Manifesto to move forwards, it's part of the geography that we are building on. We will continue to discover new and better ways of working and it looks increasingly like technical practices and cultural capabilities are too important to leave out of the deal.

# Lean

| Lean and Continuous Delivery Principles | Build quality in | Work in small batches | Computers/Humans | Continuous Improvement | Everyone is responsible |
|-----------------------------------------|------------------|-----------------------|------------------|------------------------|-------------------------|
| Eliminate waste                         |                  | ✅                     | ✅                |                        |                         |
| Amplify learning                        |                  | ✅                     |                  | ✅                      |                         |
| Decide as late as possible              |                  | ✅                     |                  |                        |                         |
| Deliver as fast as possible             |                  | ✅                     | ✅                |                        |                         |
| Empower the team                        |                  |                       |                  | ✅                      | ✅                       |
| Build integrity in                      | ✅                |                       | ✅                |                        |                         |
| See the whole                           | ✅                |                       |                  | ✅                      |



## Conclusion

Close off the post by restating the main points of the post, share any closing thoughts, and invite feedback.

## Learn more

- [link](https://www.example.com/resource)

Happy deployments! 
