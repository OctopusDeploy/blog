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

With DevOps and continuous delivery gaining traction, are the principles behind Lean and Agile still relevant? How do they compare to the 5 principles of continuous delivery and what do any differences mean for software development teams?

## Introduction

Throughout the 1990s, a revolution was brewing within the software development industry. The early phased models had divided the delivery process into skill-based steps, with designs and documents being used to run approval processes to control the project.

The process for delivering software was heavyweight and slow, with little indication that the overheads reduced the risks they were intended to mitigate.

A collection of lightweight methods was emerging, all designed to generate fast feedback and iteratively adapt the plan based on new information. Small batches, rather than controls and approvals, were being used to control the risk and to provide frequent opportunities to change direction.

The Agile Manifesto emerged From this adaptive and lightweight methods community in 2001, with Lean Software Development arriving a couple of years later.

Both Agile and Lean provided a *values and principles* based approach to software development. You could test whether an approach was Agile or Lean by testing it against a set of core statements that defined the philosophy behind each approach.

![The evolution of software delivery from phased, to adaptive, to continuous](software-delivery-history-lean-agile.png)

Continuous delivery emerged from the Lean and Agile community and provided the most concrete set of practices and capabilities to achieve the goal of delivering software frequently and safely. Apart from Extreme Programming, the majority of software development methods dealt with requirements management and communication structures. Instead, continuous delivery takes on the rest of the value stream and provides a pathway to implement an efficient and reliable delivery pipeline.

InfoQ publishes an annual report on software development culture and methods, which is inspired by Geoffrey Moore's book *Crossing the Chasm*. Moore famously visualizes a technology adoption lifecycle as a bell curve spread across 3 standard deviations.

The chart is divided into five groups with different attitudes to new technology:

- Innovators (-3)
- Early adopters (-2)
- Early majority (-1)
- Late majority (+1)
- Laggards (+3)

The early majority and late majority represent two-thirds of the adoption lifecycle, but for a technology (or in the case of InfoQ, an idea) to move into the majority it must cross the chasm from the early adopters to the mainstream majorities.

![The adoption of culture and methods for software development](infoq-culture-and-methods-2022.jpg)*Source: [InfoQ Software Development Culture and Methods 2022](https://www.infoq.com/articles/culture-trends-2022/)*

While adaptive modes of software delivery are now embedded in the late majority, the capabilities needed for continuous modes of delivery are still making their way across the chasm. Some practices (such as DevSecOps) have made the leap, while other key elements, like team topologies and empowered teams, remain at the early adopter stage.

So, with DevOps and continuous delivery gaining traction, it's a great time to compare the 5 principles of continuous delivery with the 12 Agile principles and the 7 Lean principles.

## Comparing principles

Agile, lean, and continuous delivery each provide a set of principles. This provides a basis for a comparison of their fundamental structures.

There are other techniques and capabilities in Lean and continuous delivery that don't have comparable elements in Agile, as The Agile Manifesto was designed as a broad litmus test of the agility of other specific methods, such as Extreme Programming, Scrum, and Disciplined Agile; so it didn't define further details itself.

Lean expands on its principles by describing techniques that cover the whole software development process with advice on planning, design, testing, and maintenance.

Continuous delivery uses the deployment pipeline (from the code commit to the software running in production) to focus efforts to improve the flow of change. Specific technical capabilities are described in detail to enable the *early and continuous delivery of valuable software*.

This means the principles are the only directly comparable elements of these three approaches.

### Continuous delivery principles

There are [five principles](https://continuousdelivery.com/principles/) of continuous delivery.

1. Build quality in
2. Work in small batches
3. Computers perform repetitive tasks, people solve problems
4. Relentlessly pursue continuous improvement
5. Everyone is responsible

![The relationship between the continuous delivery principles](cd-principles.png)

The principles can be arranged into a reinforcing relationship, where the first three principles act as drivers of continuous improvement, for which everyone involved is responsible.

### Agile principles comparison

In general, the principles of continuous delivery are well aligned to [The Agile Manifesto](https://agilemanifesto.org/principles.html).

![Mapping agile and continuous delivery principles](agile-continuous-delivery.jpg)

For example, working in small batches widely understood to help achieve several of the Agile principles:

- Our highest priority is to satisfy the customer through early and continuous delivery of valuable software.
- Welcome changing requirements, even late in development. Agile processes harness change for the customer's competitive advantage.
- Deliver working software frequently, from a couple of weeks to a couple of months, with a preference to the shorter timescale
- Working software is the primary measure of progress.
- Agile processes promote sustainable development. The sponsors, developers, and users should be able to maintain a constant pace indefinitely.
- Simplicity--the art of maximizing the amount of work not done--is essential.

The phrasing of the principle "work in small batches" is concise and easy to follow in practice. This continuous delivery principle contributes to the attainment of half of the Agile principles.

There is also an amplification effect as all 5 principles are combined, to "deliver working software frequently" you need to work in small batches, increase automation, and build quality in.

There are two Agile principles that are not covered by continuous delivery:

 - Business people and developers must work together daily throughout the project, and
 - The most efficient and effective method of conveying information to and within a development team is face-to-face conversation

Continuous delivery is centered on the delivery pipeline, which means the process of identification, inception, and initiation aren't a direct concern of continuous delivery. Additionally, in the front matter of the *Continuous Delivery* book, the authors highlight that the book was created according to the ideas it contained. The contributors and editors worked using different tools and from different locations. They used version control and builds to form a delivery pipeline for the book. They didn't work together on a daily basis or convey information face-to-face.

As an industry, many organizations who hadn't attempted to work remotely are now ready to experiment with distributed working and this will further challenge real-time face-to-face co-ordination as asynchronous communication becomes more common and familiar.

In terms of word counts, the principles of continuous delivery are conveyed in 21 words, rather than the 180 words of the Agile principles. A combination of hindsight and incisive editing has given us quick compass to check our direction, with the detail being provided in the specific practices of continuous delivery.

> It’s easy to get bogged down in the details of implementing continuous delivery — tools, architecture, practices, politics — if you find yourself lost, try revisiting these principles and you may find it helps you refocus on what’s important. - [Jez Humble](https://continuousdelivery.com/principles/)

You don't have to throw out The Agile Manifesto to move forwards, it's part of the geography that continuous delivery and DevOps are building on. The industry will continue to discover new and better ways of developing software. It is increasingly certain that technical practices and cultural capabilities are a key part of this emerging knowledge.

### Lean principles comparison

The Lean principles are already brief and actionable. There is complete alignment between Lean and continuous delivery, which no un-mapped areas.

![Mapping lean and continuous delivery principles](lean-continous-delivery.jpg)

With Agile, continuous delivery offered a concise alternative of 10 of the Agile principles. In the case of Lean, continuous delivery aligns completely with the principles. Each approach provides a different perspective that applies to different segments of the organization.

In an organization that has fully adopted both Lean and continuous delivery, the principles would be useful to different groups. Lean principles and techniques are useful as leadership tools and continuous delivery provides an approach specific to the individual contributors delivery efforts.

For example, with continuous delivery, automation is the practical attack vector for eliminating waste, delivering quickly, and building integrity into the system.

## Conclusion

Continuous delivery provides a concise set of principles that align to 10 of the 12 Agile principles. Continuous delivery also complements Lean, providing a different perspective on achieving similar goals.

All three sets of principles are consistent with modern management philosophy and provide different ways to review and re-focus your software delivery efforts.

You can use the continuous delivery principles to guide your software delivery efforts and refer back to Agile and Lean principles to find further inspiration for your improvement efforts.

## Learn more

- [The importance of continuous delivery (blue paper)](https://www.example.com/resource)
- CD series links
- Free tool links

Happy deployments! 
