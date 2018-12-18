---
title: Introducing our first release of Octopus Server with long-term support 2018.10 LTS
description: Octopus Server 2018.10 LTS is the first release with six months of long-term support. We recommend this release for most self-hosted customers.
author: michael.noonan@octopus.com
visibility: private
bannerImage: blogimage-ltsrelease.png
metaImage: blogimage-ltsrelease.png
published: 2018-12-19
tags:
- New Releases, LTS
---

![Cars on slow lane and fast lane](blogimage-ltsrelease.png)

Octopus Server `2018.10 LTS` is the first release in our long-term support program. It comes with six months long-term support and we recommend this release for most self-hosted customers. Customers using Octopus Cloud are always in the [fast lane](/blog/2018-09/long-term-support/index.md#fast-and-slow-lanes) where we take care of upgrades on your behalf, and you get early access to the latest features.

Learn more about our long-term support program in our [previous blog post](/blog/2018-09/long-term-support/index.md). This post has a [Q&A section](/blog/2018-09/long-term-support/index.md#qa) to help you understand how our releases fit best into your organisation.

!toc

## What features does 2018.10 LTS have?

Octopus Server `2018.10 LTS` is essentially the same as `2018.9`. It comes with (workers and worker pools)[/blog/2018-07/octopus-release-2018.7.md], [Kubernetes support](/blog/2018-10/octopus-release-2018.9/index.md), and everything we've done up until today including a small mountain of performance and security work.

## When should I install 2018.10 LTS?

There is no time like the present! You can upgrade to Octopus Server `2018.10 LTS` any time from now. We've had thousands of customers upgrade smoothly to `2018.9` and the upgrade to `2018.10 LTS` is cut from the same code. We are all taking some time off over the holiday season for a well deserved rest, hopefully just like you! That being said, we will have support staff working through the holidays in case you need our help!

## Should I avoid upgrading and stay on my current release?

Please upgrade! Our preference for your benefit and ours is that you keep your Octopus Server up to date. You benefit with the highest quality, best performing, most secure Octopus Server, and we benefit by having a smaller distribution of Octopus Server versions to support. It's a win-win situation!

- If you are using Octopus Cloud, we take care of updates on your behalf, you're always in the fast lane with early access to the newest features.
- If you are using self-hosted Octopus Server, we recommend using releases with long-term support.
  - When we ship a patch you should patch your Octopus Server - it will be less risk to upgrade than to leave your Octopus Server unpatched.
  - When we ship a new release with long-term support, you can choose to stay on your current version as long as it is still covered by our long-term support program, but we recommend keeping up with the current release where possible.
- If you are using self-hosted Octopus Server, but decide to stay in the fast lane, we highly recommend staying current just like Octopus Cloud. This gives us the best opportunity to support you. If you don't want to stay current like Octopus Cloud, perhaps the releases with long-term support are a better option for your scenario.

## When you ship a patch should I install it?

Absolutely yes! We will use this rule of thumb when deciding what to include in a patch for releases with long-term support: **Installing a patch should be safer than not installing that patch.** We will include something in an LTS patch when, for example:

- We discover a security vulnerability which will result in us raising a CVE report.
- We discover a show-stopping bug where there is no viable workaround.
- We discover an issue which is only present in a current LTS release.
- We discover something which just makes good business sense to patch.

When we discover something like this we will ship a patch for any releases still covered by long-term support.

We will not:

- Ship hundreds of LTS patches - we want stability and a high signal to noise ratio.
- Ship new features in LTS patches.
- Ship breaking changes in LTS patches.

## Should I stay in the fast lane instead of installing 2018.10 LTS and moving to the slow lane?

This is a decision you will have to make based on your scenario. We generally recommend self-hosted customers choose releases with long-term support.

Choose the **slow lane releases with long-term support** if this sounds like your scenario:

- "We prefer stability over having the latest features."
- "We upgrade Octopus about every three months."
- "We evaluate Octopus in a test environment before upgrading our production installation."

You should choose the **fast lane releases** if this sounds like your scenario:

- "We want the latest and greatest features and really fast turnaround on small enhancements and bug fixes."
- "We want to engage closely with the Octopus team, so we can help them build the best automation tooling in the world!"

## Will you still support me even if I don't upgrade?

Absolutely yes! Staying current with Octopus Server releases is mutually beneficial, but we will support any Octopus customer to the best of our ability regardless of which release you are running. If we fix a bug on your behalf, you will need to upgrade to get the bug fix anyhow. Staying current is in everyone's best interests!

## Wrapping up

Long term-support (LTS) for Octopus Server has arrived, and you can bank on it. Happy long-term deployments!