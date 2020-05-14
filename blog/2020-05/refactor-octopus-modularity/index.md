---
title: "Refactoring Octopus: Modularity, Calamari and Sasahimi"
description: Learn more about how our engineering team refactored the Octopus code base to introduce modularity, reduce complexity and eat some sashimi.
author: rob.pearson@octopus.com
visibility: private
published: 2020-05-27
metaImage: 
bannerImage: 
tags:
 - Engineering
---

// TODO: Add blog post graphic

## Why Refactor Octopus Deploy

We shipped Octopus Deploy 1.0 nearly 8 years ago and the product has grown tremendously in popularity as well as feature set. As a part of this journey, the code base has undergone some big changes including huge overhauls and rewrites. In this blog series, we will share some of the problems this growth has brough and how we're refactoring the code base to make simplify it and make it easier to change.

In this article, I talk about how we're introducing modularity to make it easier to integration with external services and tools.

## Octopus, Tentacle and Calamari! 

![Octopus and Tentacle diagram](octopus-and-tentacle.png)

It's time for walk down memory lane for a brief history lesson. Octopus has long made it easy to ship web sites and services but the logic that powered the deployment execution has moved around over the years. 

In the Octopus 1.x and 2.x timeframe, the logic to execute deployments lived in Tentacle. The Octopus Server did the main coordination and the actualy deployments were executed directly within our Tentacle agent running on deployment targets. If we shipped a new version of Octopus or an update to deployment execution, we had to ship a new version of Tentacle. They were tightly coupled and needed to be in sync.

Then in Octopus 3.0, we introduced Calamari which we called little slices of Octopus. This new component was a standalone deployment execution engine and we envisioned we'd created multiple indepedent calamari slices for different purposes and technologies. We also introduced SSH deployment targets and Tentacle mirror'd its role in that it became a simple pipe used to communicate with the Octopus Server and transfer deployment data. As time passed, Calamari grew to be a bit of a monolith and we only really had two slices: Standard Calamari and Calamari Cloud.

Over the years, this has worked well and helped our customers execute tens of millions of deployments however it has introduced some challenges. If we want to add a new integration for a new technology, there is deep knoweldge required for numerous touch points across the Octopus code based from the front end, some server side code as well as Calamari updates. WHile this works it slows us down and it's friction for us to add new integrations. We knew we needed a change and we've been planning it for a while. 

## Introducing Calamari Flavours

Our first step was to start splitting up Calamari as we originally intended. Split it into little slices with their own separate projects and distinct entry point. We still have our core Calamari for common deployment work like script execution but we're introducing additional "flavours" like Terraform Calamari, AWS Calamari, Azure Calamari etc. It's interesting to note that with each flavour, it brings support for specific platforms like Windows, macOS, Linux and ARM.

This approach brings a number of advantages:
* Each slice is totally independent and can be upgraded independently and customised as per the project's needs.
* Allows us to separate older technologies like Azure Cloud Services and keep them separate and stable while we focus on other priorities. 

## Introducing Sashimi

This leads us to introducing Sashimi which we are calling little slices of Octopus Server. Whereas Calamari flavours are little slices of our deployment execution engine, Sashimi is little slices of Octopus Server. We are taking a knife to Octopus and carving out the technology specific things. This enables us to split out the web UI and server side logic into separate self-contained components.

This is a huge step forward as this allows us to make the Octopus Server (both front-end and web services) a world-class coordination engine without any specific knowledge or hooks to technology specific steps. For example, we can have support for Azure, AWS or Terraform without the server having any specific knowledge or references of these technologies.

Our goal for this work is to simplify the development to add support for new technologies and services. In other words, developers on our team can write code to add support in isolation from Octopus, grok it and test it in isolation. This removes the mental weight of understanding all the touches points within Octopus Server. The outcome is faster integration with less headaches. i.e. We could introduce Pulumi support following the patterns created by Terraform in a very short period of time. This also opens up the possibility for third parties to add support for their technologies within Octopus.

### Sashimi components and one big problem

We're building Sashimi slices as NuGet packages. Each Sashimi package contains: 

- Zipped UI files that are injected into the main user interface.
- Server-side processing components 
- Any third party libraries or components (dependencies)
- Calamari components i.e. The standalone executables for each platform. 

One important thing to note is that this is bundling a lot of files and this can add up to a lot of storage. For example, if we had 10 Calamari Flavours and 3 platforms each. This is 30 calamari components (self contained executables) which are 40 MB each. The end result of this is that we were facing 1.2 GB of additional data per Octopus release. Obviously, this approach wasn't ideal. 

### Sashimi and calamari consolidation

To solve this storage problem, we needed to come up with a thoughtful solution. The way we overcome it is that crafted a build time process to consolidate the Calamari components and extract all common files and executables into a separate ZIP file which can the be compressed further. The common component is only 150 MB and the rest of the files are negligable. We also build an index at this stage and then we use this at runtime we can reconsistute the the appropriate components within Octopus Server and then the appropriate Calamari is transferred to the deployment tareet.

## Conclusion

With Calamari and Sashimi, we are realising our goal of little slices of Octopus and true modularity. This brings a ton of benefits but the most prominent is a simpler code base and less friction to integrate with new techologies. We're still in the middle of this transition but it's already proving it's value. None of these ideas are new but we're pleased to prioritise it and actually make it happen. 

This is a refactoring at it's best. We're improving the design of our code base and it should be transparent to our customers. 