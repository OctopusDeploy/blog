---
title: Deprecating authentication extensions
description: Find out why and when Octopus is deprecating support for custom authentication extensions.
author: robert.erez@octopus.com
visibility: private
published: 2022-11-22-1400
metaImage: blogimage-deprecatingcustomauthproviders-2022x2.png
bannerImage: blogimage-deprecatingcustomauthproviders-2022x2.png
bannerImageAlt: 125 characters max, describes image to people unable to see it.
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

Back in Octopus 3.5 we [introduced the concept of Server Extensibility](https://octopus.com/blog/octopus-deploy-3.5#octopus-deploy-server-extensibility). This was partly because we couldn't fully cater for all the different authentication mechanisms provided by Active Directory and support various other authentication providers that our customers were asking for. 

At the time, opening the authentication pipeline for customization as a win-win; our customers could ensure the system worked precisely as required for their organization, and we could relieve some pressure supporting all the different mechanisms required. 

We also saw it as an opportunity to re-architect how some of the core dependencies were included in the product, to simplify the development process. By breaking up the Octopus Server responsibilities, the thinking was that we could decouple parts of the system from one another, allowing a faster development cycle.

Unfortunately, as we've grown, some of these well-intentioned assumptions haven't gone to plan. In attempting to isolate Octopus dependencies out into separate libraries to open source authentication, we made things more difficult for ourselves without necessarily even making things easier for customers.

## Problems 

When contracts for an extension library become publicly available for consumption, you commit yourself to the maintenance and continued functionality of that contract. This means architectural decisions made early on, which may have been right at the time, can come back and bite you down the track when you need to deliver new functionality that changes those assumptions. 

Due to the way the library dependency hierarchy is structured, changes to some of our base libraries used by the Octopus Server code require customers to rebuild their assemblies just to continue using them. This adds unnecessary and oftentimes frustrating steps to what should be a simple Server upgrade process. These pain points have also made development internally slower rather than faster since the tight coupling between code still exists, just split across libraries. 

Although making the authentication providers _open for review_ was a reasonable move (and one that we continue to make for various libraries through our GitHub repositories), the way they're _open for extension_ has resulted in more problems than what they solved. So, we decided to reconsider our current Server extensibility model.

## What’s changing

If you haven't built and installed custom authentication extensions for your Octopus Server instance, then these changes don't impact you. You can stop reading now and continue upgrading without any changes required.

If you have built your own authentication assemblies from a direct fork of one of our libraries, then are some important changes you need to make so authentication continues working in new versions of Octopus Server from 2022.4. In future builds of Octopus Server, these work-arounds may also stop working.

### Enable with environment variable

New environment variables need to be set with a value of `false` for the server to ignore similar built-in authentication providers, so they don't conflict with the extension you're loading. These environment variables need to be accessible to the process running Octopus Server, which ensures that Octopus Server correctly uses your extension rather than the standard built-in ones. Details for these variables can be found in our [docs](https://docs.octopus.com).

### Updated identity

If interacting with the configuration of custom providers via the API, they'll be exposed with a different ID. The ID of all custom authentication providers will include an `authentication-custom-` prefix rather than just `authentication-`. For example, the configuration for the built-in Active Directory authentication exposed from the `/api/configuration` endpoint will be `authentication-directoryservices`, but the configuration for an extension forked off that will appear as `authentication-custom-directoryservices`. This lets us load both the built-in and custom extensions side-by-side during the transition process.

### Side impact - custom route handlers

A side-effect of supporting authentication extensibility is the ability to inject a custom HTTP handler to respond to API requests. With the initial changes, this ability will be unaffected, however this capability will also be removed when the final deprecation changes are merged. 

## What to expect next

To improve our delivery process and strengthen the safety checks put into place around the authentication providers, we plan to fully deprecate the ability to BYO authentication into Octopus Deploy by the end of 2023. 

Between now and the end of next year, we'll continue to support extensions where possible, and review how we can consolidate the custom changes our customers have created into the official providers themselves. 

Based on feedback so far, we anticipate that most customizations are only improvements to the existing providers, however we'll investigate the impacts for users with completely custom integrations. Please let us know in the comments below if that includes you.


### Get in touch if the changes impact you

Some of you may have implemented an entirely bespoke authentication mechanism. I you can't move to one provided, we want to better understand your requirements so we can work towards resolving missing capabilities. 

Get in touch with us via [support@octopus.com](mailto:support@octopus.com]), or reply to this post to let us know if this will affect your instance.

## Conclusion

We don’t take deprecation lightly at Octopus. We pride ourselves on an impressive capability of backward compatibility across a long history of our software. However, at times we need to reconsider supporting features that are causing more problems than they solve for both us and our customers. 

As we move away from the authentication extensibility model, we hope to use the next 12 months to reduce the impact this may cause for anyone currently relying on it.

Happy deployments!