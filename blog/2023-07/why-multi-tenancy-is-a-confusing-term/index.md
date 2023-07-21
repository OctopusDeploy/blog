---
title: Why 'multi-tenancy' can be a confusing term and what that means to Octopus
description: Andy looks at why 'multi-tenancy' is a tricky thing to define and how that affected Octopus's messaging
author: andrew.corrigan@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: blogimage-whatismultitenancy-2023_1.png
bannerImage: blogimage-whatismultitenancy-2023_1.png
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - tag
---

The term 'multi-tenancy' has been around for a long time and applies to many aspects of the tech industry. As we explored in our post, [what is multi-tenancy?](updatelink), the definition can depend on your industry, style of delivery, and system architecture. Ask several people what it means and you'll likely get different answers.

The most common examples of multi-tenancy see software split by:

- **Customer** - Software as a Service (SaaS) applications where each customer gets the same product but with individual resources.
- **Location** - You serve customers or locations worldwide and need to make adjustments for regional differences.
- **Hosting** - Where you deliver your software to many hosting solutions, like the cloud, data centers, on-premises, or hybrid. 
- **Business model** - You serve brick and mortar locations, like retail stores, hospitals, hotels, schools, or more.

However, modern development makes the term blurrier. When researching that last post, I thought about how many applications could easily fit into more than one or even all these categories.

That got me thinking...

## At what level do you even define your 'multi-tenancy'?

When I joined Octopus as a technical content creator, my tech knowledge was mostly from a support background rather than a development one.

Learning how communication methods differ between customer-facing tech and developer-facing tech was... eye-opening. Not least due to the weird and wonderful ways the software industry names the tools and philosophies it creates for itself.

Finding out, for example, that 'Continuous Delivery' and 'Continuous Deployment' were disparate concepts (but also not really - [we wrote about that](https://octopus.com/devops/continuous-delivery/what-is-continuous-deployment/)) and folk in DevOps shortened both to 'CD' was baffling to me, a relative outsider.

The term 'multi-tenancy' follows a similarly confusing path, given the many scenarios described further up the page. These are, of course, simplified slices of a much bigger picture, but when one app can hit more than one category, where do you draw the line?

Take Microsoft 365 (M365). M365 is Microsoft's classic Office suite modernized and delivered in a SaaS format. M365 users can access the suite through traditional desktop installs and cloud versions on almost any device.

Microsoft sells M365 to everyone:

- Personal users
- Businesses
- Enterprises (usually through resellers)

That's 3 intended customer types for the product, each delivered differently. We could call M365 a multi-tenancy service here at its highest level. At least based on our earlier definitions.

However, each scenario could also be multi-tenancy in its own right. After all, Personal M365 is a different product from the Professional versions. Personal users get their own cloud space and resources, so you could consider each customer a tenant.

M365 Business clients get an entire manageable instance of M365 and can assign individual access to staff under their company banner. In this scenario, you could also consider each *business* a tenant too.

Some businesses even have *multiple* instances of Business M365. My last employer had 2: one for head office staff and one for branch staff. Guess what we called this setup internally? You guessed it: multi-tenancy.

I haven't even touched on how delivery methods for updates differ between the desktop and cloud application versions. What about narrowing it down to a platform of choice, like Windows or MacOS? What about each application included in M365?

It's likely that Microsoft doesn't refer to much of this as 'multi-tenancy' internally (though M365's API does authenticate against a tenant). Regardless, each scenario fits nicely into one of multi-tenancy's common descriptions. M365 is layers upon layers of structures you could easily label as multi-tenancy all the way down.

And that's just one service. At what point does 'multi-tenancy' have so many definitions that it stops meaning anything at all?

That caused some internal debate for us recently, so let's talk about that.

## What multi-tenancy means to Octopus

As a deployment tool, Octopus can help you deliver your software no matter the flavor of multi-tenancy your software uses. If you call it 'multi-tenancy', it's very likely Octopus will make your deployments faster and more reliable at scale.

The bloated nature of the term 'multi-tenancy', however, posed a few challenges as we redeveloped our website and refined our messaging. Given there are so many ideas on what 'multi-tenancy' is, it's tricky to explain Octopus's benefits in a simple way recognizable to everyone.

Instead, we decided to focus on the one thing that unites all these examples of multi-tenancy. It just happens that one thing is also Octopus's specialty: the deployment process.

A common strategy to deliver software to multi-tenancy applications in all the examples above is to use 'tenanted deployments'.

Done manually, or in other deployment solutions, tenanted deployments cause duplication and manual processes. Duplication and manual processes introduce risks - risks that cause delivery rates to slow.

These are the problems that Octopus's Tenant's feature solves for most multi-tenancy instances. So that's what it means to us.

## Conclusion

'Multi-tenancy' is a term that may have had humble, singular beginnings, but has become almost abstract in modern development. It now means so many things that it almost means nothing without clarification.

We could explain Octopus's value to multi-tenancy better when we freed ourselves from the term and focused on the problems it introduces, rather than the methodologies themselves.

That's why we're moving our focus from 'multi-tenancy' to the bit we actually help you with; tenanted deployments.

## What's next

See Octopus's [Tenants use-case page](https://octopus.com/use-case/tenanted-deployments) for more information on how Octopus can help you deploy your multi-tenancy applications faster and more reliably.

Happy deployments!