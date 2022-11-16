---
title: Deprecating authentication extensions
description: The why and when of Octopus deprecating support for custom authentication extensions.
author: robert.erez@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - Engineering
  - Trust and Security
  - Integrations
  
---

Octopus Server is deprecating support for BYO authentication mechanisms. This post will talk through why this is happening and, if you are using a custom-built authentication provider, what you need to do once the changes arrive.

## Octopus Server Extensibility
Back in Octopus 3.5 we [introduced the concept of Server Extensibility](https://octopus.com/blog/octopus-deploy-3.5#octopus-deploy-server-extensibility). This was partly driven by our realization that we were unable to fully cater for all the different authentication mechanisms provided by Active Directory, as well as support various other authentication providers that our customers were asking for. At the time we saw the opening up of the authentication pipeline for customisation as a win-win; our customers could ensure the system worked in a way that was precisely required for their org, and we could relieve some pressure to support all the different mechanisms that would be required. 

As an added benefit, we saw it as an opportunity to re-architect the way that some of the core dependencies are included in the product, ostensibly making the development process simpler. By breaking up the Octopus Server responsibilities, the thinking was that we could decouple parts of the system from one another allowing a faster development cycle.

Unfortunately, as we have grown, some of these well intentioned assumptions have not borne the fruit we had intended. In attempting to isolate Octopus dependencies out into separate libraries in order to open source authentication, we inevitably made things more difficult for ourselves without necessarily even making things easier for customers.

## Problems 
As soon as contracts for an extension library become publicly available for consumption, you commit yourself to the maintenance and continued functionality of that contract. This means that architectural decisions made early on which may have been right at the time could come back and bite you down the track when you need to deliver new functionality that changes those assumptions. 

Due to the way the library dependency hierarchy is structured, changes to some of our base libraries utilized by the Octopus Server code require users to rebuild their assemblies just to continue using them. This adds unnecessary and oftentimes frustrating steps to what should be an otherwise simple Server upgrade process. These pain points have also made development internally slower rather than faster since the tight coupling between code still exists, just split across libraries. 

Although making the authentication providers _open for review_ was a reasonable move (and one that we continue to make for various libraries through our GitHub repositories), the way in which they were made _open for extension_ have resulted in more problems than what they solved. As such we have decided to reconsider our current Server extensibility model.

## What’s Changing
If you have never built and installed custom authentication extensions for your Octopus Server instance then these changes will have no impact on you. You can stop reading now and continue upgrading without any changes required.

For anyone who has built their own authentication assemblies from a direct fork of one of our libraries then there will be some important changes you need to make to allow authentication to continue working in new versions of Octopus Server from 2022.4. In future builds of Octopus Server, these work-arounds may also stop working.

### Enable with Environment Variable
New environment variables will need to be set with a value of `false` in order for the server to ignore similar built-in authentication providers so that they do not conflict with the extension you are loading. These environment variables will need to be accessible to the process running Octopus Server which ensures that Octopus Server correctly uses your extension rather than the standard built-in ones. Details for these variables can be found in our [docs](https://docs.octopus.com).

### Updated Identity
If interacting with the configuration of custom providers via the API, they will now be exposed with a different Id. The Id of all custom authentication providers will now include an `authentication-custom-` prefix rather than just `authentication-`. For example, the configuration for the built-in Active Directory authentication exposed from the `/api/configuration` endpoint will be just `authentication-directoryservices` but the configuration for an extension forked off that will appear as `authentication-custom-directoryservices`. This allows us to load both the built-in and custom extensions side-by-side during the transition process.

### Side Impact - Custom Route Handlers
One side-effects of the changes made to support authentication extensibility is the ability to inject a custom HTTP handler to respond to api requests. With the initial changes this ability will be unaffected however this capability will also be removed when the final deprecation changes are merged. 

## What to expect next
In order to improve our delivery process and strengthen the safety checks put into place around the authentication providers, we plan to fully deprecate the ability to BYO authentication into Octopus Deploy by the end of 2023. Between now and the end of next year, we intend to continue to support extensions where possible, and review where possible how we can consolidate the custom changes our customers have created into the official providers themselves. Based on feedback so far, we anticipate that most customisations have been simply improvements to the existing providers however we will investigate what the impacts will be for any users with completely custom integrations. Please let us know if that includes you.


### Call to action
Some of you may have implemented an entirely bespoke authentication mechanism and if you are unable to move to one of the provided ones, we would be interested in understanding better your requirements so that we can work towards resolving missing capabilities. Get in touch with us via support@octopus.com, or reply to this post to let us know if this will affect your instance.

## Conclusion
We don’t take deprecation lightly at Octopus, and pride ourselves on an impressive capability of  backwards compatibility across a long history of our software. Having said that, at times we need to reconsider supporting features that are causing more problems than they solve for both us and our customers. As we move away from the Authentication Extensibility model, we hope to use the next 12 months to reduce the impact this may cause for anyone currently relying on it.

Happy deployments!