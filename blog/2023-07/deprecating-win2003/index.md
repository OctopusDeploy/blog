---
title: Dropping support for Windows 2003
description: Octopus will be dropping support for Windows 2003 Targets in 2024
author: robert.erez@octopus.com
visibility: private
published: 2026-06-05-1400
metaImage: na
bannerImage: na
bannerImageAlt: na
isFeatured: false
tags: 
- Product
---

Octopus Deploy will drop support for Windows 2003 Targets from the 2024.1 release. This change marks the start of a much more structured and hopefully predictable process to the way in which Octopus Deploy announces and executes our platform support policy.

## Why?
Windows Server 2003, a twenty year old Operating System, was marked as end of life by Microsoft over eight years ago in 2015. With no further security updates or patches being made available the recommendation from Microsoft has for a long time been to migrate to a newer supported OS. Despite the Microsoft themselves dropping support, it may come as a surprise to a lot of our customers that Octopus Deploy currently still officially supports Windows 2003 deployment targets. This requires a complecated architecture and compilation of [Calamari](https://octopus.com/docs/octopus-rest-api/calamari), our remote execution engine. 

Not only are we doing a disservice to our customers by not encouraging them to upgrade their systems, in doing so we hold back our own codebase which is required in many cases to cater for this lowest common OS denominator.

Although there may always be a long tail of customers requiring deployments to machines which are not running the latest version of the various supported platforms, it is still important that the guidance we provide has some alignment with what the platform vendors themselves suggest. 


## What Dropping Target Support Mean?                                                
All software has a lifecycle, and end of support for an Operating System in Octopus Deploy means that we will no longer develop or test for the usage of that platform during standard workloads. This may mean degraded functionality best or it could be a complete inability to run tasks on those platforms. 

In the case of Windows Server 2003, the deprecation in `2024.1` will likely be coupled with some improvements made to parts of our execution system that render the execution of complex deployments to those servers generally unavailable. 

On the rare cases that Octopus deprecates functionality we first must consider what sort of impact it will have on our users. We rely on telemetry from installed Octopus instances and conversations with our customer success team to help shape these decisions and provide insights on usage. Unsurprisingly in this case, our metrics indicate that only a very very small handful of customers still have a Tentacle running Windows Server 2003 and most of those represent a single target.

For this reason as well as the technical constraints imposed on us as described above, the drop of support of Windows Server 2003 will take effect from the 2024.1 version of Octopus Server, expected for release in early 2024. 

                                                                                                    
## The effects of Dropping Support

This depreciation will take place in 2 stages. 

### Pre 2024.1
From the next 2023.3 release, and in the most recently patched 2023.2 release, health checks and deployments that occur on targets detected as running these soon-to-be deprecated platforms will begin generating warnings. The goal is to ensure customers are aware that they are running machines which could soon be affected by the changing support. Since this change will also allow our tooling to upgrade from .net40 to .net462, we will also log a warning if at least .net462 or greater is not detected on your Windows targets.

### Post 2024.1
Tooling changes introduced in the 2024.1 release will mean that Windows Server 2003 machines are no longer expected to function and deployments will likely fail due to the unsupported .net frameworks involved. From this point Octopus Deploy will consider Windows Server 2003 an unsupported platform for rich deployment pipelines due to its reliance on these tools. 

Customers who still require workloads to run on Windows Server 2003 will have a set of limited options at their disposal for continuing to perform their tasks.

Although our primary recommendation is that customers upgrade to an Operating System within the vendor’s support, that option might not always be available. Understandably some customers might not be ready to upgrade their targets to later Windows Operating Systems or for technical reasons unable to move away from that platform. There are still options available for these cases which will provide some capabilities, albeit in a limited capacity.

Shard Your Octopus Instance - Octopus licensing allows more than one active instance at a time. Customers who need to run deployments on these older unsupported targets may wish to install a secondary Octopus Server instance with a version prior to 2024.1 where this capability is supported. Our LTS strategy currently allows support for Octopus Instance up to 12 months past their release date so users can expect to continue to receive security patches or major bug fixes through this period. Once that LTS period is over however, even this instance 
Raw Scripting
Don’t Upgrade - If you are unable to continue to upgrade 

## Deprecations In Future
Octopus is aiming to make the support schedule more predictable by announcing and triggering target deprecations in the first release of the calendar year. We want to encourage users to keep their targets up to date with vendor supported platforms and will shortly be realigning our target policies with this in mind. 

### Summary
Although Microsoft themselves have dropped support for Windows Server 2003 several years ago, Octopus Deploy has continued to invest in ensuring customers who rely on this OS can deploy to it. Unfortunately supporting these outdated platforms has a cost which not only is no longer viable for us to continue and so
