---
title: Deprecating authentication extensions
description: Find out why and when Octopus is deprecating support for custom authentication extensions.
author: robert.erez@octopus.com
visibility: public
published: 2022-11-22-1400
metaImage: blogimage-deprecatingcustomauthproviders-2022x2.png
bannerImage: blogimage-deprecatingcustomauthproviders-2022x2.png
bannerImageAlt: A stylized Octopus and shield icon
isFeatured: false
tags: 
  - Product
  - Engineering
---

Octopus Server is deprecating support for BYO authentication mechanisms. 

In this post, I explain why this is happening and, if you're using a custom-built authentication provider, what you need to do when the changes arrive.

## Octopus server extensibility

In Octopus 3.5 we [introduced the concept of server extensibility](https://octopus.com/blog/octopus-deploy-3.5#octopus-deploy-server-extensibility). This was primarily driven by customer demand, allowing our customers to adapt Octopus’ authentication mechanisms to their unique requirements. 

At the time, opening the authentication pipeline for customization was a win-win; our customers could ensure the system worked precisely as required for their organization, and we could relieve some pressure supporting all the different mechanisms required. 

We also saw it as an opportunity to re-architect how some of the core dependencies were included in the product, to simplify the development process. By breaking up the Octopus Server responsibilities, we wanted to decouple parts of the system from one another, allowing a faster development cycle.

Octopus Server has grown significantly since then, and as we've grown, some of our previous assumptions haven't had the desired impact, but instead caused unwanted side-effects. In attempting to isolate Octopus dependencies out into separate libraries to open source authentication,  development across dependencies became more difficult without making things easier for our customers.

## Problems 

When contracts for an extension library become publicly available for consumption, you commit yourself to the maintenance and continued functionality of that contract. This means architectural decisions made early on, which may have been right at the time, become difficult to adapt, particularly when new functionality requires changes to that underlying system.

Due to the way the library dependency hierarchy is structured, changes to base libraries used by Octopus Server require users of these libraries to rebuild their customizations to continue using them. This adds unnecessary and oftentimes frustrating steps to what should be a simple server upgrade process. 

Having core abstractions spread across libraries and repositories also impacts development - when you want to make cross-cutting changes, they're harder to reason about and execute when the code isn't co-located in a single repository.

Although making the authentication providers _open for review_ was a reasonable move (and one that we continue to make for various libraries through our GitHub repositories), the way they're _open for extension_ no longer aligns with the technical direction we want to move in with Octopus Server. So, we decided to reconsider our current server extensibility model.

## What’s changing

If you haven't built and installed custom authentication extensions for your Octopus Server instance, then these changes don't impact you. You can stop reading now and continue upgrading without any changes required.

In the short term, custom extensions will continue to work as they have in the past. You will need to recompile your libraries to account for dependency changes, but the same mechnaism for loading them into an Octopus Server instance will remain unchanged. In future builds of Octopus Server at the end of 2023 however, all custom authentication mechanisms will stop working entirely.

**Note: An earlier version of this blog indicated that environment variables would need to be set and authentication identities changed in API calls. Recent updates to the product means that these workarounds are no longer required.**

### Side impact - custom route handlers

A side-effect of supporting authentication extensibility is the ability to inject a custom HTTP handler to respond to API requests. With these initial changes, this undocumented ability will be unaffected however this capability will also be removed when the extensions are fully deprecated. 

## What to expect next

To improve our delivery process and strengthen the safety checks for authentication providers, we plan to fully deprecate the ability to BYO authentication into Octopus Deploy by the end of 2023. 

Between now and the end of 2023, we'll continue to support extensions where possible, and review how we can consolidate the custom changes our customers have created into the official providers themselves. 

Based on feedback so far, we anticipate most customizations are only improvements to the existing providers, however we'll investigate the impacts for users with completely custom integrations. Please let us know in the comments below if that includes you.

### Get in touch if the changes impact you

Some of you may have implemented an authentication mechanism forked off one of our libraries, and some may have developed an entirely custom authentication provider. If you can't move to one of the built-in mechanisms, we want to better understand your requirements so we can work towards resolving missing capabilities. 

Get in touch with us via [support@octopus.com](mailto:support@octopus.com) or comment below to let us know if this will affect your Octopus instance.

## Conclusion

We don’t take deprecation lightly at Octopus. We pride ourselves on strong backwards compatibility across a long history of our software. However, at times, we need to reconsider supporting features that are no longer in the best interest of our customers or Octopus Server. 

As we move away from the authentication extensibility model, we hope to use the next 12 months to reduce the impact this may cause for anyone currently relying on it.

Happy deployments!
