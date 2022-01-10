---
title: Introducing long-term Support for Octopus Server
description: In Q4 2018 we will be shipping our first release of Octopus Server with long-term support.
author: michael.noonan@octopus.com
visibility: public
bannerImage: blogimage-ltsrelease.png
bannerImageAlt: Cars on slow lane and fast lane
metaImage: blogimage-ltsrelease.png
published: 2018-09-17
tags:
- Company
---

![Cars on slow lane and fast lane](blogimage-ltsrelease.png)

**Heads up: The LTS program launched in January 2019. Learn more about [how the LTS program affects you](https://octopus.com/docs/administration/upgrading/long-term-support).**

We are planning to ship our very first release of Octopus Server with **long-term support (LTS)** in Q4 2018. This post explains our plans for Octopus Server in an LTS world, and how you will benefit. We also explore some of the background to this decision and some of the mechanics we'll use behind the scenes.

We think a lot of self-hosted customers will want to predominantly use Octopus Server releases with long-term support, but if you use self-hosted and want to keep right up to date with everything we're doing, you can keep enjoying that cake too!

!toc

## Introducing the Octopus Server LTS Program

We will ship a new release of Octopus Server with **six months of long-term support** on a **three month cadence**. This means there will be **two current LTS releases** at any point in time.

Each LTS release will roll up all the features and bug fixes we've stabilized during that three month period. A release of Octopus Server with long-term support will:

- Get critical bug fixes and security patches for up to six months.
- Not get new features, minor enhancements, nor minor bug fixes - these will roll up into the next LTS release.

### Patching

When it comes to deciding what to include or exclude from a patch we will use this rule of thumb:

> Installing a patch should be safer than not installing that patch.

You should use the same rule of thumb when deciding whether to install a patch onto your Octopus Server.

### Announcements

We will announce each new LTS release of Octopus Server in a blog post with the `LTS` tag, clearly stating which releases are still covered by long-term support, and releases where long-term support has expired.

Each release of Octopus Server will clearly indicate if it is an LTS release on the [downloads page](https://octopus.com/downloads) and inside the product itself.

## The Power to Choose

We realize not every customer is the same. We want to make it really easy for anyone to answer this question: _What is the best release of Octopus Server to install?_

We want to give you the power to choose, along with simple guidance to help make an informed choice.

### Introducing the Fast and Slow Lanes {#fast-and-slow-lanes}

Under the covers, we plan to keep working the same way we have for the last several years: shipping bug fixes and minor enhancements with a quick turnaround and working closely with our customers to design and test new features. What we are adding is a special release cadence, where the release is based on the most stable version at that point in time with some additional quality assurance, complete with the offer of six months long-term support.

Internally we think about this as two "release lanes":

- The **fast lane** is exactly what we do today. We ship new features when they are ready, usually every 4-6 weeks, and ship bug fixes and minor enhancements into patches every few days.
- The **slow lane** is where we will stabilize and ship releases with long-term support, along with any patches containing critical bug fixes and security patches for up to six months.

#### Octopus Cloud is in the Fast Lane

[Octopus Cloud](https://octopus.com/cloud) customers will be using releases from the fast lane: we make the choice for you. You will get the latest and greatest features as soon as they are ready, along with the quickest turnaround time on bug fixes and small enhancements.

#### Self-hosted: The Power to Choose

Self-hosted customers can decide for themselves. We recommend choosing a lane and sticking with it, but you can [switch lanes](#switching-lanes) when it suits your situation.

Choose the **slow lane releases with long-term support** if this sounds like your scenario:

- "We prefer stability over having the latest features."
- "We upgrade Octopus about every three months."
- "We evaluate Octopus in a test environment before upgrading our production installation."

You should choose the **fast lane releases** if this sounds like your scenario:

- "We want the latest and greatest features and really fast turnaround on small enhancements and bug fixes."
- "We want to engage closely with the Octopus team, so we can help them build the best automation tooling in the world!"

## Q&A {#qa}

I've covered the broad details of our long-term support program, and I'll answer some common questions here. If you have any questions at all, please feel free to ask in the comments!

### How Do You Choose What to Include in an LTS Patch?

We will use this rule of thumb: **Installing a patch should be safer than not installing that patch.** We will include something in an LTS patch when, for example:

- We discover a security vulnerability which will result in us raising a CVE report.
- We discover a show-stopping bug where there is no viable workaround.
- We discover an issue which is only present in a current LTS release.
- We discover something which just makes good business sense to patch.

We will not:

- Ship hundreds of LTS patches - we want stability and a high signal to noise ratio.
- Ship new features in LTS patches.
- Ship breaking changes in LTS patches.

### Can We Move Between the Slow and Fast Lanes? {#switching-lanes}

Yes, you can switch lanes in a controlled fashion. "Accelerating" to a fast lane release will result in you running a higher version of Octopus Server - it's just a normal upgrade. If you would like to "decelerate" back to the slow lane releases with long-term support, just wait until the next LTS release is shipped and upgrade to that release.

### Will You Maintain the Monthly Cadence for Fast Lane Releases?

We currently ship releases on an approximate monthly cadence. We think a predictable cadence is more important for customers using the slow lane releases with long-term support - it will help them plan their upgrades.

- In the **slow lane** we will aim to ship LTS releases on a strict three month cadence.
- In the **fast lane** we will aim to ship new releases on a monthly cadence, however, sometimes we may decide to ship a fast lane release earlier, or take some extra time to harden a fast lane release before shipping it.

If you prefer a more predictable cadence choose the slow lane releases with long-term support.

### Will You Change Your Versioning Strategy?

Not really, no. We will pick the next release number for each release of Octopus Server [just like we do today](/blog/2018-01/version-change-2018.md) with some extra context:

- We will add `LTS` to some part of the version for releases which come with long-term support.

## Wrapping Up

We are introducing long-term support (LTS) for Octopus, and you can bank on it.

If you have any concerns or questions, please reach out in the comments below!
