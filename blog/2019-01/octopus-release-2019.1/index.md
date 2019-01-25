---
title: Spaces - Octopus Deploy 2019.1
description: Spaces
author: nick.josevski@octopus.com
visibility: private
published: 2019-01-29
metaImage: blogimage-spaces.png
bannerImage: blogimage-spaces.png
tags:
 - New Releases
 - Spaces
---

`// TODO: Add Video link` 

## Focus on what matters to you with Spaces

Octopus Deploy is proud to ship [Spaces](https://octopus.com/spaces). Our goal with Spaces is to help teams organise their Octopus servers better and focus on the projects, environments and deployments that are important to them. Reduce the noise and work more efficiently. 

You can [download Octopus Server 2019.1](https://octopus.com/downloads) now!

## In this post

!toc

## Give teams their own space

Spaces is a simple concept that has a big benefit for everyone. It allows teams to group the projects, environments, tenants, step templates and other resources into a Space. 

Our updated navigation bar lets you easily jump between spaces. Instead of seeing dozens or hundreds of unrelated projects and other Octopus resources, you'll just see the resources for that space.

## Put team leads in control with improved permissions

As a part of building Spaces, we needed to revamp our security and permissions. The end result of this is that our security model is easier to manage and teams can now manage the security of the space indepedently. This can eliminate the need to submit requests to system admins.

Our updated security model is flexible so you can customize it as needed.

## Breaking Changes

Using Spaces is entirely opt-in. We have done extensive work and testing to ensure the feature is as backwards compatible as possible.

In order to support Spaces and to deliver improvements to how you can [configure teams](https://octopus.com/blog/team-configuration-improvements) we made some breaking API changes. Before upgrading you should [review the full set here](https://octopus.com/downloads/compare?from=2018.12.1&to=2019.1.0).

## Integrations / Tentacle

Once you have updated to 2019.1 and want to start making use of the Spaces feature in your Octopus instance you will need to upgrade your integrations including [Tentacle](https://octopus.com/downloads).

## Upgrading

As usual [steps for upgrading Octopus Deploy](https://octopus.com/docs/administration/upgrading) apply. Please see the [release notes](https://octopus.com/downloads/compare?to=2019.1.0) for further information. Self-Hosted Octopus customers can [download](https://octopus.com/downloads/2019.1.0) the latest release now. For Octopus Cloud, you will start receiving the latest bits in about 2 weeks during their maintenance window.

That's it for this month. Feel free to leave us a comment and let us know what you think! Go forth and deploy!

## Learn More

- [Spaces home page](https://octopus.com/spaces)
- [Spaces blog series](https://octopus.com/blog/octopus-spaces-blog-series-kick-off)
- [Spaces documentation](https://g.octopushq.com/spaces)
- [Spaces and Workes Webinar](https://hello.octopus.com/webinar-spaces-workers/on-demand?utm_referrer=http%3A%2F%2Foctopus.com%2Fblog%2Foctopus-release-2019.1)