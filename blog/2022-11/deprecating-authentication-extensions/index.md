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
  - Trust and Security
  - Integrations
---

Octopus Server is deprecating support for BYO authentication mechanisms. 

In this post, I explain why this is happening and, if you're using a custom-built authentication provider, what you need to do when the changes arrive.

## Octopus server extensibility

In Octopus 3.5 we [introduced the concept of server extensibility](https://octopus.com/blog/octopus-deploy-3.5#octopus-deploy-server-extensibility). This was primarily driven by customer demand, allowing our customers to adapt Octopus’ authentication mechanisms to their unique, specific requirements. 

At the time, opening the authentication pipeline for customization was a win-win; our customers could ensure the system worked precisely as required for their organization, and we could relieve some pressure supporting all the different mechanisms required. 

We also saw it as an opportunity to re-architect how some of the core dependencies were included in the product, to simplify the development process. By breaking up the Octopus Server responsibilities, we wanted to decouple parts of the system from one another, allowing a faster development cycle.

Octopus Server has grown significantly since this initial thinking was done and as we've grown, some of these assumptions we had previously made have not had their desired impact, and have instead caused other unwanted side-effects. In attempting to isolate Octopus dependencies out into separate libraries to open source authentication,  development across dependencies became much more difficult without making things easier for our customers.

## Problems 

When contracts for an extension library become publicly available for consumption, you commit yourself to the maintenance and continued functionality of that contract. This means architectural decisions made early on, which may have been right at the time, become very difficult to adapt, particularly when new functionality requires changes to that underlying system.

Due to the way the library dependency hierarchy is structured, changes to base libraries used by Octopus Server require users of these libraries to rebuild their customizations to continue using them. This adds unnecessary and oftentimes frustrating steps to what should be a simple Server upgrade process. 

Having core abstractions spread across libraries and repositories also impinges upon development - when you want to make cross-cutting changes, they are harder to reason about and execute when the code is not co-located within a single repository.

Although making the authentication providers _open for review_ was a reasonable move (and one that we continue to make for various libraries through our GitHub repositories), the way they're _open for extension_ no longer aligns with the technical direction we want to move in with Octopus Server. So, we decided to reconsider our current server extensibility model.

## What’s changing

If you haven't built and installed custom authentication extensions for your Octopus Server instance, then these changes don't impact you. You can stop reading now and continue upgrading without any changes required.

If you have built your own authentication assemblies from a direct fork of one of [our authentication libraries](https://octopus.com/docs/administration/server-extensibility/customizing-an-octopus-deploy-server-extension), there are some important changes listed below that you need to make so authentication continues working in new versions of Octopus Server from 2022.4. 

In future builds of Octopus Server at the end of 2023 these work arounds will also be deprecated resulting in all custom authentication mechanisms no longer working.

### Enable with environment variable

New environment variables need to be set with a value of `false` for the server to ignore similar built-in authentication providers, so they don't conflict with the extension you're loading. These environment variables need to be accessible to the process running Octopus Server, which ensures that Octopus Server correctly uses your extension rather than the standard built-in ones. Details for these variables can be found in our [docs](https://docs.octopus.com).

### Updated identity

If interacting with the configuration of custom providers via the API, they'll be exposed with a different ID. The ID of all custom authentication providers will include an `authentication-custom-` prefix rather than just `authentication-`. 

For example, the configuration for the built-in Active Directory authentication exposed from the `/api/configuration` endpoint will be `authentication-directoryservices`, but the configuration for an extension forked off that will appear as `authentication-custom-directoryservices`. 

This lets us load both the built-in and custom extensions side-by-side during the transition process.

### Side impact - custom route handlers

A side-effect of supporting authentication extensibility is the ability to inject a custom HTTP handler to respond to API requests. With the initial changes, this ability will be unaffected, however this capability will also be removed when the extensions are fully deprecated. 

## What to expect next

To improve our delivery process and strengthen the safety checks for authentication providers, we plan to fully deprecate the ability to BYO authentication into Octopus Deploy by the end of 2023. 

Between now and the end of 2023, we'll continue to support extensions where possible, and review how we can consolidate the custom changes our customers have created into the official providers themselves. 

Based on feedback so far, we anticipate most customizations are only improvements to the existing providers, however we'll investigate the impacts for users with completely custom integrations. Please let us know in the comments below if that includes you.

### Get in touch if the changes impact you

Some of you may have implemented an authentication mechanism forked off one of our libraries and some may have developed an entirely custom authentication provider. If you can't move to one of the built-in mechanisms, we want to better understand your requirements so we can work towards resolving missing capabilities. 

Get in touch with us via [support@octopus.com](mailto:support@octopus.com), or reply to this post to let us know if this will affect your instance.

## Conclusion

We don’t take deprecation lightly at Octopus. We pride ourselves on strong backwards compatibility across a long history of our software. However, at times, we need to reconsider supporting features that are no longer in the best interest of our customers or Octopus Server. 

As we move away from the authentication extensibility model, we hope to use the next 12 months to reduce the impact this may cause for anyone currently relying on it.

Happy deployments!
