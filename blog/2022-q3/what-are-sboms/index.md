---
title: "What are SBOMs?"
description: Find out what SBOMs are, learn about the US government's Executive Order, why SBOMs are important, and how Octopus can help you.
author: terence.wong@octopus.com
visibility: public
published: 2022-09-21-1400
metaImage: placeholderimg.png
bannerImage: placeholderimg.png
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags:
  - DevOps
  - Cloud Orchestration
  - Testing
---

If you're a developer or you manage an enterprise software application, you may have been asked about the components in your application. Why do people want to know? Customers want to trust your application, they want your application to be secure. Enterprise vendors and government bodies also want to know because they're concerned with security issues for their users using your software.  

Software applications are made up of several sources, open-source, in-house, or a mixture of both. As the list of dependencies grows, how can an application be secure if the individual components used to build the application aren't known?

A bill of materials lists the required inventory to produce a given output reliably. Bills of materials have been used for years to provide transparency and repeatability to the manufacturing process. Software bills of materials (SBOMs) apply a similar concept to bills of materials to software. SBOMs itemize the components in a software application in a list that developers can share across teams.

## Executive Order on Improving the Nation's Cybersecurity

On May 12, 2021, The United States government released an Executive Order on [Improving the Nation's Cybersecurity](https://www.whitehouse.gov/briefing-room/presidential-actions/2021/05/12/executive-order-on-improving-the-nations-cybersecurity/). 

> The Federal Government must bring to bear the full scope of its authorities and resources to protect and secure its computer systems, whether they are cloud-based, on-premises, or hybrid.  The scope of protection and security must include systems that process data (information technology (IT)) and those that run the vital machinery that ensures our safety (operational technology (OT)).

The Executive Order is trying to minimize the cybersecurity risk in the supply chain when people acquire software. Risk increases as the number of unknown components in the software applications increases. 

The Executive Order requires all software bought by the US government to produce an SBOM. This has several implications for business inside and outside the US. Any US government project now has to produce SBOMs for security purposes. Any vendor that can't produce SBOMs for their products won't be approved to work with government projects. 

The Order is likely the beginning of many similar orders to require SBOMs worldwide. As awareness of SBOMs increase, it's probable businesses will begin to demand SBOMs. If you work in software, SBOMs are probably in your future.

## What goes into SBOMs?

The National Telecommunications and Information Administration (NTIA) provides guidelines on constructing an SBOM. NTIA conducted a proof of concept of SBOMs in healthcare, which informed the [baseline elements required for an SBOM](https://ntia.gov/files/ntia/publications/howto_guide_for_sbom_generation_v1.pdf). 

The baseline elements are summarized here:

- Author Name - The author of the SBOM document describing the Primary component. The author may not be the same as the supplier of the Primary component.
- Supplier Name - the supplier of a component.
- Component Name - the name of the component.
- Version String - the version of the component.
- Component Hash - a cryptographic hash used to identify the binary instance of a component.
- Unique Identifier - a unique identifier for a component. Multiple identifiers may exist for an element because different systems may use anoter identifier.
- Relationship - is used to establish that a component includes another component. In addition, Relationship is used to document knowledge about the completeness of the list of components included in another component.
- Component Relationships
- Primary Component – the component described by the SBOM.
- Included Component – the components included in another component.

## Why are SBOMs important?

The requirement for SBOMs has a significant impact on software. Software is built collaboratively and often contains several third-party libraries that use other third-party libraries. Without the ability to generate SBOMs, software won't be compliant with the Executive Order. 

Government bodies and organizations acting under the Executive Order need to choose software that can produce an SBOM on demand and can prove that each component is not a cybersecurity risk. The widespread use of SBOMs will increase the trust between vendors and government bodies.

Software vendors need a reliable way to detect any known vulnerabilities in their deployed application. SBOMs let you be proactive in addressing vulnerabilities, reducing the likelihood your tools can be hacked. Vulnerability scanning also helps you avoid awkward conversations with customers who scan your applications themselves and report vulnerabilities.

The requirement for SBOMs can be seen as just an additional step in the build process to generate the artifact, but it does raise some questions, such as:

- How do you pair an SBOM to a deployable artifact so one can be matched to the other?
- How do you know what version of your application is in production so you can scan the associated SBOM in the weeks or months after the application was deployed?
- How do you orchestrate the deployment of your application and publish the associated SBOM?
- How do you schedule SBOM scanning to proactively detect newly discovered vulnerabilities?
- How do you scan old SBOM versions to identify previous releases of your software that include vulnerable components?

At Octopus, we built a free tool that answers all of those questions for you and meets your SBOM requirements.

## The Octopus Workflow Builder - how it helps with SBOM requirements

The [Octopus Workflow Builder](https://octopusworkflowbuilder.octopus.com/#/) helps you generate SBOMs and build them into your deployment process. The tool demonstrates how SBOM files are constructed as part of the build, and then demonstrates how Octopus Deploy can scan the SBOM as part of the deployment. With the Builder, you're provided with a sample project demonstrating how deployable artifacts and their associated SBOMs are tightly coupled in a release, allowing you to orchestrate and publish the SBOM with any application deployment, schedule SBOM scanning, or access old SBOM versions.

## Conclusion

In 2021, the US government issued an executive order to improve the nation's cybersecurity. The executive order mandated that software components need to be known to the government to minimize security risks. If you work in software, this requires you to expose the components of your applications or risk being froze out of government IT related projects. You can make your software components known through SBOMs. SBOMs are a list of components in a software application that is sharable and generated automatically on each application release. To help you with this requirement, Octopus Deploy has developed a free tool that produces SBOMs as part of the build and scans it as part of the deployment.  

Happy deployments!