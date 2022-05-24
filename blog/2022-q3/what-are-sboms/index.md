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

If you are a developer, or if you manage an enterprise software application, you may have been asked to produce a list of components in your application. Customers want to know what goes into your applcation because they want it to be free of vulnerabilties. Enterprise vendors and government bodies also want to know because they are concerned with security concerns for their users using your software.  

How can an application be secure when the individual components used to build the application are not known? 

A bill of materials is a manufacturing term that lists the required inventory to produce a given output reliably. Bills of materials have been used for years to provide transparency and repeatability to the manufacturing process. Software bills of materials (SBOMs) apply a similar concept to bills of materials to software. SBOMs itemize the components in a software application in a list that developers can share across teams.

## Executive Order

On the 12th of May 2021, The United States government released an executive order on [Improving the Nation's Cybersecurity](https://www.whitehouse.gov/briefing-room/presidential-actions/2021/05/12/executive-order-on-improving-the-nations-cybersecurity/). In the executive order, 

> The Federal Government must bring to bear the full scope of its authorities and resources to protect and secure its computer systems, whether they are cloud-based, on-premises, or hybrid.  The scope of protection and security must include systems that process data (information technology (IT)) and those that run the vital machinery that ensures our safety (operational technology (OT)).

The executive order seeks to minimize the cybersecurity risk in the supply chain that arises from acquired software. Cybersecurity risk increases as the number of unknown components in the software applications increases. The executive order requires developers to produce an SBOM for all applications developed in the US. This has several implications for business inside and outside the US. Any US government project will now have a requirement to produce SBOMs for security purposes. Any vendor that cannot produce SBOMs for their products are unlikely to be approved to work with government projects. The executive order is likely the beginning of many similar orders to require SBOMs worldwide. As awareness of SBOMs increase, the more likely businesses anywhere will begin to demand SBOMs be made availalble. If you work in software, SBOMs will likely be in your future.

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

The requirement for SBOMs has a significant impact on open-source software. Open-source software is built collaboratively and contains several third-party libraries that use other third-party libraries. Without the ability to generate SBOMs, open-source software will not be compliant with the executive order. The inability to generate SBOMs also affects proprietary software that uses some open-source software in production. Government bodies and organizations that are acting under the executive order are obligated to choose software that can produce an SBOM on demand and can prove that each component is not a cybersecurity risk. The widespread use of SBOMs will increase the trust between vendors and governmental bodies.

Software vendors need a reliable way to detect any known vulnerabilities in their deployed application. SBOMs allow developers to be proactive in addressing vulerabilities, reducing the likelihood that their tools can be hacked. Vulnerability scanning will also allow you to avoid some awkward conversations with customers that scan your applications themselves and report vulnerabilities.

The requirement for SBOMs could be seen as just an additional step in the build process to generate the artifact, but it does raise some questions like:

- How do you pair an SBOM to a deployable artifact so one can be matched to the other?
- How do you know what version of your application is is production so you can scan the associated SBOM in the weeks or months after the application was deployed?
- How do you orchestrate the deployment of your application and publish the associated SBOM?
- How do you schedule SBOM scanning to proactively detect newly discovered vulnerabilities?
- How do you scan old SBOM versions to identify previous releases of your software that include vulnerable components?

## Octopus Builder - How it can help with your SBOM requirements

Software applications can have thousands of different dependencies. Manually listing each component is not sustainable as developers can replace or upgrade components frequently. Automation is necessary to supply SBOMs accurately and quickly. Octopus has produced a free tool called Octopus Builder that helps developers make SBOMs. The tool generates an SBOM as part of the build, and Octopus Deploy can scan the SBOM as part of the deployment. The tool dramatically reduces the need to figure out how to piece together SBOMs from separate sources on the internet. You can find Octopus Builder on [our website]()

## Conclusion

In 2021, the US government issued an executive order to improve the nation's cybersecurity. The executive order mandated that software components need to be known to the government to minimize security risks. If you work in software, this requires you to expose the components of your applications or risk being froze out of government IT related projects. You can make your software components known through SBOMs. SBOMs are a list of components in a software application that is sharable and generated automatically on each application release. To help you with this requirement, Octopus Deploy has developed a free tool that produces SBOMs as part of the build and scans it as part of the deployment.  


Happy deployments!
