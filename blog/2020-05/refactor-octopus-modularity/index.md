---
title: "Refactoring Octopus: Introducing Modularity"
description: Learn more about how our engineering team refactored the Octopus code base to introduce modularity, reduce complexity and eat some sashimi.
author: rob.pearson@octopus.com
visibility: private
published: 2020-05-27
metaImage: 
bannerImage: 
tags:
 - Engineering
---

// TODO: Add Article Graphic

BLUF

## Why Refactor Octopus Deploy

History lesson. Introduction to deployment logic. Tentacle first. If we updated Octopus, Tentacle had to be updated too. Thye were tightly coupled.

Then we introduced Calamari. Little slices of Ocotpus. Calamari was owned by Octopus Server and pushed down to Tentacle. So whenever you upgraded Octopus, you could run the latest code for your step. Nice improvement.

We ended up with two slices. Standard Calamari and Calamari Cloud. It's a big monolith built one or two ways. It didn't come out as intended.

What we started with Modularity 
We split this up into little slices. 

// TODO: Graphic w/ Terraform Calamari, AWS Calamari, Azure Calamri, IIS Calamari etc.

Each calamari is it's own separate project w/ distinct entry points. Totally independent and can be upgraded indenpendently/customised per project.

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