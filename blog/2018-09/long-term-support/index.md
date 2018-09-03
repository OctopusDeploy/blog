---
title: Introducing long-term support for Octopus Server
description: In Q4 2018 we will be shipping our first release of Octopus Server with long-term support.
author: michael.noonan@octopus.com
visibility: private
bannerImage: 
metaImage: 
published: 2018-09-08
tags:
- New Releases, LTS
---

We are planning to ship our very first release of Octopus Server with **long-term support (LTS)** in Q4 2018. This post will explain our plans for Octopus Server in an LTS world and how you will benefit. We will also explore some of the background to this decision and explore some of the mechanics we'll use behind the scenes.

We think a lot of self-hosted customers will want to predominantly use Octopus Server releases with long-term support, but if you want to keep right up to date with everything we're doing you can keep enjoying that cake too!

!toc

## Introducing the Octopus Server LTS program

We will ship a new release of Octopus Server with **six months of long-term support** on a **three month cadence**. This means there will be **two current LTS releases** at any point in time.

Each LTS release will roll up all the features and bug fixes we've stabilized during that three month period. A release of Octopus Server with long-term support:

- will get critical bug fixes and security patches for up to six months
- will not get new features, minor enhancements, nor minor bug fixes - these will rolled up into the next LTS release

### Patching

When it comes to deciding what to include or exclude from a patch we will use this rule of thumb:

> Installing a patch should be safer than not installing that patch.

You should use the same rule of thumb when deciding whether to install a patch onto your Octopus Server.

### Announcements

We will announce each new LTS release of Octopus Server in a blog post with the `LTS` tag, clearly stating which releases are still covered by long-term support, and releases where long-term support has expired.

Each release of Octopus Server will clearly indicate if it is an LTS release on the [downloads page](https://octopus.com/downloads), and inside the product itself.

## The power to choose

We realise not every customer is the same. We want to make it really easy for anyone to answer this question: _What is the best release of Octopus Server to install?_

- For customers on Octopus Cloud, we make the choice for you. _Yes, you are our test subjects, and we love you for it!_
- For some self-hosted customers the answer will be clear: **We depend on Octopus deeply and want the most stable release possible. We want the latest release with long-term support!**
- For other self-hosted customers the answer will be different: **We really want to explore the Octopus support for Kubernetes. Let's install that release the day it comes out!**

We want to give you the power to choose, along with simple guidance to help make an informed choice.

### Introducing the fast and slow lanes

Under the covers, we plan to keep working the same way we have for the last several years: shipping bug fixes and minor enhancements with a quick turnaround, and working closely with our customers to design and test new features. What we are adding is a special release cadence, where the release is based on the most stable version at that point in time with some additional quality assurance, complete with the the offer of six months long-term support.

Internally we think about this as two "release lanes":

- The **fast lane** is exactly what we do today. We ship new features when they are ready, usually every 4-6 weeks, and ship bug fixes and minor enhancements into patches every few days.
- The **slow lane** is where we will stabilize and ship releases with long-term support, along with any patches containing critical bug fixes and security patches for up to six months.

In practical terms:

- Octopus Cloud customers will be using releases from the fast lane
- Self-hosted customers will be able to choose the releases they want to install
  - The [downloads page](https://octopus.com/downloads) will default to releases from the slow lane with long-term support

## Q&A

I've covered the broad details of our long-term support program, and I'll answer some common questions here. If you have any questions at all please feel free to ask in the comments!

### How do you choose what to include in a LTS patch?

We will use this rule of thumb: **Installing a patch should be safer than not installing that patch.** For example:

- We discover a security vulnerability which will result in us raising a CVE report
- We discover a show stopping bug where there is no viable workaround
- We discover an issue which is only present in a current LTS release
- We discover something which just makes good business sense to patch

We will not:

- Ship hundreds of LTS patches - we want stability and a high signal to noise ratio
- Ship new features in LTS patches
- Ship breaking changes in LTS patches

### Can we move between the slow and fast lanes?

Yes, in a controlled fashion. "Accelerating" to a fast lane release will result in you running a higher version of Octopus Server - it's just a normal upgrade. If you would like to "decelerate" back to the slow lane releases with long-term support, just wait until the next LTS release is shipped and upgrade to that.

### Will you change your versioning strategy?

Not really, no. We will just pick the next release number for each release of Octopus Server [just like we do today](/blog/2018-01/version-change-2018.md) with some extra context:

- We will add `LTS` to some part of the version for releases which come with long-term support.

Based on our current plans, our release schedule should look something like this:

- `2018.7` shipped in July 2018 primarily with support for workers - [see release notes](/blog/2018-07/octopus-release-2018.7.md)
- `2018.8` shipped in September with support for multiple packages in steps and Kubernetes support in alpha - [see release notes](/blog/2018-09/octopus-release-2018.8/index.md)
- `2018.9` is scheduled to ship in late September 2018 with the Kubernetes support in full release
- `2018.10 LTS` is scheduled to ship in early October 2018 and will be based on the most reputable release of `2018.9.x` (including Kubernetes support, excluding Spaces because that comes in `2018.11.x`)
- `2018.11` is scheduled to ship in late October and will be the first installment of Spaces