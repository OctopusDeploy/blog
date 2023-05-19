---
title: Cloud nomad architecture
description: As more organizations rethink their cloud and microservices decisions, it's time for cloud-nomad architectures.
author: steve.fenton@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 
isFeatured: false
tags: 
  - DevOps
---

Over the past 5 years, organizations have been quietly rethinking their approach to cloud hosting and microservice architectures.

The cloud offers a seemingly infinite ability to scale. However, this can lead to unpredictable spending, especially with volatile costs such as egress charges. At the same time, many organizations are finding the cost of complexity with microservices outweighs the benefits.

Rather than reverting to the old state, the organizations are pushing forward based on the hard lessons learned over the last decade. To take advantage of the new reality, you'll need to apply cloud-nomad architectures.

This article explains what they are and why they are needed.

## Cloud-native architecture

You may be familiar with cloud-native architecture, which encourages application design that maximizes the benefits of cloud environments. To be *cloud-native* you'll use technologies like containers, microservices, and immutable infrastructure to build loosely-coupled systems that are easy to manage and operate.

The [Cloud Native Computing Foundation (CNCF)](https://www.cncf.io/) promotes cloud-native computing and manages over 100 projects that help you achieve it. Their most famous project is Kubernetes.

In the rush to the cloud, powered by microservices rocket fuel, many organizations have let their context fade into the background. With the boosters cooling down, it's time to recognize the power of gravity.

Enter, cloud-nomad architecture.

## What is cloud-nomad architecture?

Ancient hunter-gatherers learned to move to where the food was. Rather than having a fixed home, they would migrate based on seasonal availability of water, plants, and animals. In more modern times, tinkers and traders moved to where they could find new customers.

To maintain mobility, nomads developed portable dwellings or temporary shelters like [goahti](https://en.wikipedia.org/wiki/Goahti), [tipis](https://en.wikipedia.org/wiki/Tipi), and [wickiups](https://en.wikipedia.org/wiki/Wigwam).

The ability to easily move to a new location is central to cloud-nomad architecture.

The CNCF definition of cloud-native encompasses public and private clouds. You should be able to run your cloud-native application on the public cloud, in a data centre, on using on-prem infrastructure. Taking this a step further, to say that it should be easy to move between these options, gives us cloud-nomad architecture.

To achieve this, you must avoid depending on vendor-specific features and embrace ephemeral infrastrcuture. You need infrastructure automation that works across different hosting scenarios.

Cloud-nomad architecture also encourages you to balance your microservice architecture against Conway's Law. If you have 5 teams in your organization and 100 microservices, the complexity will likely outweigh the benefits.

:::hint
Mel Conway wrote a 1967 paper titled [How Do Committees Invent](http://www.melconway.com/Home/Committees_Paper.html), where he made a social observation that can be summarized as:

> Any organization that designs a system (defined broadly) 
> will produce a design whose structure is a copy of the 
> organization's communication structure.

Fred Brooks shared this idea in *The Mythical Man Month*, and named it *Conway's Law*. You can find this book in our [DevOps reading list](https://octopus.com/devops/reading-list/#the-mythical-man-month-book).
:::

A cloud-nomad architecture has all the properties of cloud-native architecture. Additionally:

- It is minimally complex, 
- It values portability by avoiding vendor-specific dependencies
- It automatically provisions infrastructure in a vendor-agnostic way

Like nomadic shelters, your application should be easy to pack up, move, and set up in a new location. When the shelter is too heavy, has too many parts, or depends on finding highly specific replacement parts it becomes less portable.

## Teams and microservices

A key benefit of a service-oriented architecture is it lets a team work independently without tripping over the work of other teams. Other teams consume a service based on a well-defined interface (usually an API), which means the internal code can be easily changed without conflicts with work being done elsewhere.

Small teams using trunk-based development often don't encounter the problems microservices solve. When you have fewer than 10 developers all committing code many times a day, you won't experience code conflicts and the knock-on effects of a large merge.

As your development organization grows, it gets harder to introduce change at the same velocity without generating pain for developers. By splitting code along team boundaries, you get to keep the benefits of small teams in exchange for a little coordination cost.

If you worked in a small e-commerce company, you might scale by splitting both your development team and your software architecture to create:

- A product search team who manages a search API
- E-commerce team who manages the website

If you've read Team Topologies, you'll recognize these as stream-aligned teams.

When you don't match the architecture and team designs, you start to create problems at a faster rate than the benefits accrue. You may not see the problems at a factor of 2x services to teams, but as the ratio increases you'll soon come unstuck.

To reduce the downside of large numbers of services, you can design complexity controls. For example, [Monzo used service isolation](https://monzo.com/blog/we-built-network-isolation-for-1-500-services) to reduce the number of links between their 1,500 services. However, in most cases, organizations eventually realize they have been fighting Conway's Law and seek to reduce the number of services.

### Moving away from microservices

Some of the early pioneers of microservices have started backing up in recent years. Some are moving to a monolithic architecture, and others are reducing the number of services.

When an organization moved from a monolith to microservices, and back again, it's likely they ended up with a better-designed monolith. Just because the code is in once place doesn't mean it can't be organized or divided between many teams.

A successful monolith incorporates strong isolated units of code without the performance overheads of making external calls or the complexity of managing versions of APIs.

An alternative to the full monolith approach is to move from many tiny services to fewer larger services. This has been termed macroservices, mescoservices, or plain old service-oriented architecture. The goal of this design is to get better aligned to Conway's Law and remove unnecessary complexity and reduce performance problems.

::hint
What should we call appropriately sized services?

In social psychology, macro analysis is "monolithic". It studies large social units, like global and national systems. Micro analysis falls at the other end of the scale. It's concerned with individuals, partners, and households. Mesco analysis refers to the study of things of moderate size, like communities.

Although *mescoservices* would be the most apt, it's likely *macroservices* will become the software industry term.
:::

For example, [Gergely Orosz](https://lobste.rs/s/mc3k1c/at_uber_we_re_moving_many_our) confirmed this approach was being taken by Uber. They found that running thousands of microservices caused more long-term problems than it solved. In particular, problems emerge around:

- Security
- Monitoring
- Testing
- Maintenance
- CI/CD
- Service levels
- Keeping library versions updated

The number of services appropriate to your organization will vary You should aim to find the right balance, rather than following general trends or hype cycles.

## Moving back on-prem

There has been a recent trend of organizations moving some or all of their workloads back to on-prem datacentres. Some of these have been high-profile, such as [Basecamp](https://world.hey.com/dhh/why-we-re-leaving-the-cloud-654b47e0), but far more are happening quietly.

One of the key drives is cost, but in many cases, organizations are finding a positive performance impact from moving back to on-prem infrastructure.

Just as new monoliths have positive design elements that were learned from writing microservices, on-prem infrastructure can benefit from cloud-native developments. You don't need to abandon containers when you leave the public cloud, they can benefit your new on-prem strategy.

Whether you run on public cloud, private cloud, or on-prem, it's critical to maintain your ability to change your mind later. Being able to move easily is the key to keeping options open.

## Accepting gravity before you jump

The gravitational effect on cloud and microservices isn't new, it's the result of people experiencing the predicted effects of the over-enthusiastic adoption of a cool idea.

The fundamental reasons for using public cloud remain, such as accessing temporary resources to scale to demand for a short time. Trying to manage unpredictable loads on your own infrastructure means maintaining capacity that may rarely be used, which is expensive. Managing normal loads is often cheaper to do yourself.

Similarly, Conway's Law requires you to think about your team design. It will require the team's code to be isolated from other code, but services aren't the only way to do that. Having more services than teams indicates they are being created for some other reason. You should know what that reason is and work out if microservices are the most appropriate solution.

There has always been a tendency for organizations to jump into new concepts with both feet. It's not only developers keen to broaden their experience, some organizations like to project a modern image by adopting trendy practices and technologies.

Perhaps the general lesson is to be more skeptical of hype trains. To tie this back to the original metaphor, it's about building gravity into your model before you jump.

## Conclusion

Cloud-native architectures lend themselves well to satisfying Conway's Law with minimal complexity. A key measure of their success should be how easy it is to move your workloads to different cloud providers or back to your on-prem infrastructure.

Happy deployments!
