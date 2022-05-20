---
title: Comparing continuous delivery and Agile
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

After two decades, The Agile Manifesto is still a big influence on software development teams around the world. With DevOps and continuous delivery now gaining traction, it's a good time to compare the 5 principles of continuous delivery to the 12 Agile principles. In his book *Continuous Delivery Pipelines*, Dave Farley places "apply Lean and Agile principles" in his 7 essential techniques, so how well do they align? Let's find out.

## Agile and continuous delivery

Continuous delivery uses the deployment pipeline as a focusing lens for the continuous improvement of the software delivery process. There are specific capabilities, techniques, and principles involved and a strong preference for automation where it is appropriate. The technical capabilities of continuous delivery have been proven to have a statistical relationship to the success of an organization. A continuous delivery pipeline with fully automated approval stages is sometimes called _continuous deployment_, where no human intervention is required after the code is integrated into the main branch in version control.

Agile is a set of principles attached to a collective manifesto that emerged from the lightweight and adaptive software development movement. The Agile Manifesto was written to represent a set of values to use as a litmus test for development methods. Rather than defining a model or framework, the values and principles could guide the creation and evolution of many different approaches. Some successful examples of Agile methods include Extreme Programming, Scrum, and Dynamic Systems Development Method (DSDM).

Continuous delivery was influenced by Agile and Lean, and took it's name from the first principle of The Agile Manifesto: "Our highest priority is to satisfy the customer through early and continuous delivery of valuable software". Continuous delivery seeks to deliver the maximum value for the least effort and requires software be in a constant state of readiness. It uses automation to clear the way for fast and frequent delivery of software.

### The Agile Manifesto



People have tried to re-write the Agile principles a few times, but the original authors have left it alone.

Alistair Cockburn

 > ...it was a free-for-all, that is to say, there was no given purpose or agenda. Self-organization at it’s finest, held together by deep respect and generous listening on all sides. What emerged was what these exact 17 people could produce together at that particular moment in history. We agreed at the end never to update the manifesto for that exact reason.

https://web.archive.org/web/20170626102447/http://alistair.cockburn.us/How+I+saved+Agile+and+the+rest+of+the+world

When people rewrite it, we tend to get a well written, well thought through, single-person perspective. This is not representative of the community, though the best of these attempts have gained some momentum, such as Modern Agile, created by Industrial Logic's CEO, Joshua Kerievsky. Modern Agile places more emphasis on cultural factors.

https://modernagile.org/

https://heartofagile.com/

Agile describes something of a standard for a method to obtain - many different methods can be created and tested against the manifesto. For example, Extreme Programming (XP) and Dynamic System Development Method (DSDM) are both specific methods that have their own principles and techniques, but are both considered Agile because they pass the requirements of each of the manifesto principles.

### Continuous delivery

Various methods contained different mixes of communication, process, and technical practices - though most were light on technical practices with XP being the notable difference.

Continuous delivery is a modern method that is focused on specific practices, with less emphasis on communication and process - however the practices have undergone intense scrutiny via research backed analysis, so we now know that they are worth the investment.

CD can be supplemented with a simple communication and process tool such as Kanban to co-ordinate the work, as it's highly complementary to many of the principles of CD and the development of a strong DevOps model for the relationships between practices (which places CD firmly in the picture) provides signposts to cultural and organizational capabilities that complement the practices of CD.

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
| Early and continuous delivery / valuable software |                  | ✅                   | ✅                |                        |                         |
| Welcome / harness change                          |                  | ✅                   |                  |                        |                         |
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


## Conclusion

Close off the post by restating the main points of the post, share any closing thoughts, and invite feedback.

## Learn more

- [link](https://www.example.com/resource)

Happy deployments! 
