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

// TODO: Graphic w/ Terraform Calamari, AWS Calamari, Azure Calamri, IIS Calamari etc.

Our first step was to start splitting up Calamari as we originally intended. Split it into little slices with their own separate projects and distinct entry point. Each slice is totally independent and can be upgraded indenpendently and customised as per the project's needs. Examples of the slices are:



This alone brings advantes 
Advantages:
- Totally independant. Customise each project based on its needs.
- We could separate an older legacy technology like Azure Cloud Services and leave it while we upgrade other projects to take advantage of the latest updates and innovations.
- Each flavour comes different platforms.

## Introducing Sashimi

Slices of Octopus Server. Carving out the technology specific things. 

- Web UI will be split out.
- Octopus Server will split out server side technology specific processing into separate sashimi projects/components.
- The Octopus will be a core coordination engine without any core specific technology bits.
- Goal: Carving these things off will simplify building technology specific support indepedent of Octopus (i.e. less development overhead). Remove the mental weight of understanding all the touches points within Octopus Server. Outocme is faster integration with less headaches. i.e. We could introduce pulumi support following the patterns created by terraform in a very short period of time vs. 

This also unlocks the ability for us to get third parties to add support within Octopus as well as us outsourcing development for specific tasks if required.

## Sashimi Structure

Built as nuget packages.
- Zipped UI files that get injected.
- Server Side logic DLL 
- Any third party DLLs (depdencies) to make the server side core work.
- Also packaged is the calamari components. The standalone executables. (Server doesn't referecne them directly).

10 Flavours. 3 platforms each. 40MB each. == Giant Octopus. 1.2 GIG and we're only going to add more flavours. 

---

This isn't a good idea. 

## Consolidated calamari platform at Build Time.

We compare all of these packages and pull out the common bits into a zip file. This way, we get great consolidation and compression and the core calamari zip file is only 150 meg. If we need a specific calamari, we recostitute it on the server and redeploy it ot the target.

## Conclusion

Summarise the benefits. The change should be transparent to the customer too.