---
title: What are SBOMs?
description: An description of SBOMs, how they are important and how Octopus helps address the problems
author: terence.wong@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: placeholderimg.png
bannerImage: placeholderimg.png
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags:
  - DevOps
  - Containers Series
  - Containers
  - Cloud Orchestration
  - Testing
  - Everything as Code
---

<!-- see https://github.com/OctopusDeploy/blog/blob/master/tags.txt for a comprehensive list of tags -->

## What are SBOMs?

Developers build software from in-house components, paid and open-source software. The wide variety of software components in an application makes it hard to track what software an application uses. The lack of traceability in software applications poses a security threat to governments and businesses. How can an application be secure when the individual components used to build the application are not known?

A bill of materials is a manufacturing term that lists the required inventory to produce a given output reliably. Bills of materials have been used for years to provide transparency and repeatability to the manufacturing process. Software bills of materials (SBOMs) apply a similar concept to bills of materials to software. SBOMs itemize the components in a software application in a list that developers can share across teams.

## Executive Order

On the 12th of May 2021, The United States government released an executive order on Improving the Nation's Cybersecurity. In the executive order, the government `acknowledges growing risks across the cybersecurity landscape and seeks to enhance the federal government's cybersecurity posture correspondingly.` The executive order seeks to minimize the cybersecurity risk in the supply chain that arises from acquired software. Cybersecurity risk increases as the number of unknown components in the software applications increases. The executive order requires developers to produce an SBOM for all applications developed in the US.

## What goes into an SBOM?

The National Telecommunications and Information Administration (NTIA) provides guidelines on constructing an SBOM. NTIA conducted a proof of concept of SBOMs in healthcare, which informed the [baseline elements required for an SBOM](https://ntia.gov/files/ntia/publications/howto_guide_for_sbom_generation_v1.pdf). The baseline elements are summarized here:

- Author Name - The author of the SBOM document describing the Primary component. The author may not be the same as the supplier of the Primary component

- Supplier Name - the supplier of a component

- Component Name - the name of the component

- Version String - the version of the component

- Component Hash - a cryptographic hash used to identify the binary instance of a component

- Unique Identifier - a unique identifier for a component. Multiple identifiers may exist for an element because different systems may use another identifier

- Relationship - is used to establish that a component includes another component. In addition, Relationship is used to document knowledge about the completeness of the list of components included in another component.

- Component Relationships
- Primary Component – the component described by the SBOM
- Included Component – the components included in another component

## Why is it important?

The requirement for SBOMs has a significant impact on open-source software. Open-source software is built collaboratively and contains several third-party libraries that use other third-party libraries. Without the ability to generate SBOMs, open-source software will not be compliant with the executive order. The inability to generate SBOMs also affects proprietary software that uses some open-source software in production. Government bodies and organizations that are acting under the executive order are obligated to choose software that can produce an SBOM on demand and can prove that each component is not a cybersecurity risk.

## How Octopus can help with the free tool

Software applications can have thousands of different dependencies. Manually listing each component is not sustainable as developers can replace or upgrade components frequently. Automation is necessary to supply SBOMs accurately and quickly. Octopus has produced a free tool that helps developers make SBOMs. The tool generates an SBOM as part of the build, and Octopus Deploy can scan the SBOM as part of the deployment. The tool dramatically reduces the need to figure out how to piece together SBOMs from separate sources on the internet.

## Conclusion

In 2021, the US government issued an executive order to improve the nation's cybersecurity. The executive order mandated that software components need to be known to the government to minimize security risks. The mechanism to make software components known is SBOMs. SBOMs are a list of components in a software application that is sharable and should be generated automatically on each application release. Octopus Deploy has developed a free App builder tool that produces SBOMs as part of the build and scans it as part of the deployment.  


Happy deployments!
