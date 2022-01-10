---
title: Introducing our first release with long-term support - Octopus Server 2018.10 LTS
description: Octopus Server 2018.10 LTS is the first release with six months of long-term support. We recommend this release for most self-hosted customers.
author: michael.noonan@octopus.com
visibility: public
bannerImage: blogimage-ltsrelease.png
bannerImageAlt: Cars on slow lane and fast lane
metaImage: blogimage-ltsrelease.png
published: 2018-12-19
tags:
- Engineering
---

![Cars on slow lane and fast lane](blogimage-ltsrelease.png)

Octopus Server `2018.10 LTS` is the first release in our [long-term support program](/blog/2018-09/long-term-support/index.md)(LTS). It comes with six months long-term support and we recommend this release for most self-hosted customers.

You can [download Octopus Server 2018.10 LTS](https://octopus.com/downloads) now!

Our LTS releases will be in the slow lane with a new LTS release every three months, all of which have six months support. We'll also continue to put out new releases roughly once a month, the releases between LTS releases will be our fast lane releases that include the latest features.

Customers using Octopus Cloud are always in the [fast lane](/blog/2018-09/long-term-support/index.md#fast-and-slow-lanes) where we take care of upgrades on your behalf, and you get early access to the latest features.

Learn more about our LTS program in our [previous blog post](/blog/2018-09/long-term-support/index.md). This post has a [Q&A section](/blog/2018-09/long-term-support/index.md#qa) to help you understand how our releases fit best into your situation.

!toc

## What features does 2018.10 LTS have?

Octopus Server `2018.10 LTS` is essentially the same as `2018.9`. It comes with [workers and worker pools](/blog/2018-07/octopus-release-2018.7.md), [Kubernetes support](/blog/2018-10/octopus-release-2018.9/index.md), and everything we've done up until today including a small mountain of performance and security work.

## Breaking changes

We are not introducing any breaking changes, however `2018.10 LTS` will be the last version of Octopus which supports upgrades from older versions of Octopus. Learn about [upgrading older versions of Octopus](https://octopus.com/docs/administration/upgrading).

## When should I install 2018.10 LTS?

There is no time like the present! You can upgrade to Octopus Server `2018.10 LTS` now. We've had thousands of customers upgrade smoothly to `2018.9` and the upgrade to `2018.10 LTS` is cut from the same code.

You can [download Octopus Server 2018.10 LTS](https://octopus.com/downloads) now!

## Should I avoid upgrading and stay on my current release?

Please upgrade! Our preference, for your benefit and ours, is that you keep your Octopus Server up to date. You benefit with the highest quality, best performing, most secure Octopus Server, and we benefit by having a smaller distribution of Octopus Server versions to support. It's a win-win situation!

- If you are using Octopus Cloud, we take care of updates on your behalf, you're always in the fast lane with early access to the newest features.
- If you are using self-hosted Octopus Server, we recommend using releases with LTS.
  - When we ship a patch, like `2018.10.1 LTS`, you should patch your Octopus Server - it will be less risk to upgrade than to leave your Octopus Server unpatched.
  - When we ship a new release with LTS, you can choose to stay on your current version as long as it is still covered by our LTS program, but we recommend keeping up with the current release where possible.
- If you are using self-hosted Octopus Server, but decide to stay in the fast lane, we highly recommend staying current just like Octopus Cloud. This gives us the best opportunity to support you. We are looking at options to make upgrades easier than ever before - reach out to our support team for help with automating your upgrades. If you don't want to stay current, in line with Octopus Cloud, perhaps the releases with LTS are a better option for your scenario.

## When you ship a patch like 2018.10.1 LTS should I install it?

Absolutely, yes! We will use this rule of thumb when deciding what to include in a patch for releases with LTS: **Installing a patch should be safer than not installing that patch.** We will include something in an LTS patch when, for example:

- We discover a security vulnerability which will result in us raising a CVE report.
- We discover a show-stopping bug where there is no viable workaround.
- We discover an issue which is only present in a current LTS release.
- We discover something which just makes good business sense to patch.

When we discover something like this we will ship a patch for any releases still covered by LTS.

We will not:

- Ship hundreds of LTS patches - we want stability and a high signal to noise ratio.
- Ship new features in LTS patches.
- Ship breaking changes in LTS patches.

## Should I stay in the fast lane instead of installing 2018.10 LTS and moving to the slow lane?

By introducing the LTS program we are giving you the power to choose between upgrading in the [fast lane or the slow lane](/blog/2018-09/long-term-support/index.md#fast-and-slow-lanes). This is a decision you will have to make based on your scenario. Learn about [switching lanes](/blog/2018-09/long-term-support/index.md#switching-lanes).

We generally recommend self-hosted customers choose releases with LTS, and by installing `2018.10 LTS` you are switching to the slow lane.

You should choose the **slow lane releases with LTS** if this sounds like your scenario:

- "We prefer stability over having the latest features."
- "We upgrade Octopus about every three months."
- "We evaluate Octopus in a test environment before upgrading our production installation."

You should choose the **fast lane releases** if this sounds like your scenario:

- "We want the latest and greatest features and really fast turnaround on small enhancements and bug fixes."
- "We want to engage closely with the Octopus team, so we can help them build the best automation tooling in the world!"

## Will you still support me even if I don't upgrade?

Absolutely, yes! Staying current with Octopus Server releases is mutually beneficial, but we will support all Octopus customers to the best of our ability regardless of which release you are running. If we fix a bug on your behalf, you will need to upgrade to get the bug fix anyhow. Staying current is in everyone's best interests!

## Wrapping up

LTS for Octopus Server has arrived, and you can bank on it. Happy long-term deployments!

## Learn more

* [Self-hosted? Download the latest version](https://hubs.ly/H0gCMqJ0)
* Blog Series: [Automating your database deployments with Octopus Deploy](https://hubs.ly/H0gCMRR0)
* [DevOps best practice: How Octopus handles rollbacks](https://hubs.ly/H0gCMRX0)
* [Announcing Octopus Cloud](https://hubs.ly/H0gCMqM0)
* Documentation: [Upgrading your Octopus](https://hubs.ly/H0gCMS40)
* Documentation: [High Availability](https://hubs.ly/H0gCMqN0)
* Documentation: [Spaces](https://hubs.ly/H0gCMSb0)