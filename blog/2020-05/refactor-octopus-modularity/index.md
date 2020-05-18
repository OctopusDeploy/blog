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

In 2012, we shipped Octopus Deploy 1.0, and nearly 8 years later, the product continues to grow in popularity as well as feature set. As a part of this journey, the code base has undergone some big changes, including significant overhauls and rewrites. This post is the first post in a new blog series, where we are sharing some of the problems of this growth and how we're refactoring the Octopus codebase to simplify it and make it easier to change.

In this article, I talk about how we're introducing modularity to make it easier to integrate Octopus with external services and tools.

## Octopus, Tentacle, and Calamari! 

![Octopus and Tentacle diagram](octopus-and-tentacle.png)

It's time to walk down memory lane for a short history lesson. Octopus has long made it easy to ship web sites and services, but the programming logic that powers deployment execution has moved around over the years. 

In the Octopus 1.x and 2.x timeframe, the deployments execution logic lived in Tentacle. Octopus Server did the primary coordination, and deployments were executed directly within our Tentacle agent running on deployment targets. If we shipped a new version of Octopus or made an update to the deployment execution logic, we had to ship a new version of Tentacle. They were tightly coupled and required to be in sync.

Then in Octopus 3.0, we introduced Calamari, which we called little slices of Octopus. This new component was a standalone deployment execution engine, and we envisioned we'd created multiple independent calamari slices for different purposes and technologies. Tentacle was simplified as it became a simple pipe to communicate with Octopus Server and securely transfer deployment data. Deployment work was delegated to Calamari and which benefited both components. As time passed, Calamari has grown to be a bit of a monolith, and we only had two slices: Standard Calamari and Calamari Cloud.

Tentacle and Calamari in this approach worked very well, and they have helped our customers execute tens of millions of deployments. It has, however, introduced some challenges. If we want to add a new technology integration, it requires the developer to have in-depth knowledge of numerous touchpoints across the Octopus codebase from the front end to the server-side code and Calamari. While this works, it slows us down, and it adds friction to add new features. We knew we needed a change, and we've been planning it for a while. 

## Introducing Calamari flavors

Our first step was to start splitting up Calamari as we originally intended, into pieces we are calling "flavors". The split is along step categories (e.g. AWS, Terraform, K8, etc). Each of these is a separate executable, which allows us to vary the platforms we target for each flavor (e.g. Windows, macOS, Linux, and ARM) as well as the runtime (e.g. .NET Core 3.1). This clean separation will allow us to more quickly iterate on a particular technology without needing to bring all the rest along for the ride.

Calamari flavors bring several advantages:
* Each slice is independent and can be upgraded independently and customized as per the project's needs.
* Allows us to separate older technologies like Azure Cloud Services and keep them separate and stable while we focus on other priorities. 

## Introducing Sashimi

With Calamari flavors conceptualized, this leads us to our next major refactoring or new concept. Introducing Sashimi, which we are calling little slices of Octopus Server. Whereas Calamari flavors are little slices of our deployment execution engine, Sashimi is little slices of Octopus Server. We are taking a knife to Octopus and carving out the technology-specific things. This change enables us to split out the web UI and server-side logic into separate self-contained components.

This is a huge step forward as this allows us to make the Octopus Server (both front-end and web services) a world-class coordination engine without any specific knowledge or hooks for technology-specific steps. For example, we can have support for Azure, AWS, or Terraform without the server having any specific knowledge or references of these technologies.

Our goal for this work is to simplify the development to add support for new technologies and services. In other words, developers on our team can write code to add support in isolation from Octopus, grok it, and test it in isolation. This update removes the mental weight of understanding all the touchpoints within Octopus Server. The outcome is faster integration with fewer headaches. i.e., We could introduce Pulumi support following the patterns created by Terraform in a very short period. It also opens up the possibility for third parties to add support for their technologies within Octopus with very little overhead.

### Sashimi components and one big problem

Each Sashimi slices is contained within a [NuGet package](https://nuget.org), and it contains: 

- Zipped UI files that we inject into the main user interface.
- Server-side processing components.
- Any third party libraries or components (dependencies).
- Standalone calamari executables for each platform. 

This approach introduced an interesting problem that we didn't expect. Each Sashimi slice bundle's numerous files, and this can add up to a lot of storage. For example, if we had 10 Calamari flavors for 3 platforms each, this produces 30 calamari components at 40 MB each. The result of this is that we were facing 1.2 GB of additional data per Octopus release. This approach wasn't ideal. 

### Sashimi and calamari consolidation

To solve this file size problem, we needed to come up with a thoughtful solution. Our solution is to use a build-time process to consolidate the Calamari components and extract all common files and executables into a separate ZIP file, which we can compress further. The common component is only 150 MB, and the rest of the files are negligible. We also build an index at build-time, which we use at runtime so we can reconstitute the appropriate components within Octopus Server. Octopus then transfers the required Calamari flavor to the deployment target.

## Conclusion

With Calamari and Sashimi, we are achieving our goal of little slices of Octopus and true modularity. This approach brings many benefits, the most prominent of which is a simpler codebase, which enables us to add new integrations more easily. We're still in the middle of this transition, but it's already proving it's value. This is refactoring at it's best. We're improving the design of our codebase, and it should be transparent to our customers. 
