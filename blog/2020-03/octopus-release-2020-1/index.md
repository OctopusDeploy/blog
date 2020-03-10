---
title: "Octopus 2020.1: Dark Mode and Better Linux Support"
description: "Octopus 2020.1: Dark Mode and Better Linux Support ... TODO"
author: rob.pearson@octopus.com
visibility: private
bannerImage: 
metaImage: 
published: 2020-03-16
tags:
- Product
---

<iframe width="560" height="315" src="https://www.youtube.com/embed/TODO" frameborder="0" allowfullscreen></iframe>

---

We're proud to ship Octopus 2020.1, our first release of the year with some great updates and benefits. 

* [Dark Mode 😎](blog/2020-03/octopus-release-2020-1/index.md#dark-mode) is cool and it's easier on your eyes.
* [Octopus and Octopus CLI are more at home on Linux and macOS](blog/2020-03/octopus-release-2020-1/index.md#octopus-and-octopus-cli-are-now-more-at-home-on-linux-and-macos) - AWS and Azure built-in steps are cross platform and you can now install and use the Octopus CLI via HomeBrew, Yum and APT. 
* [Worker pools can now scoped to environments or tenant tags](blog/2020-03/octopus-release-2020-1/index.md#worker-pools-can-be-scoped-by-environment-and-tenants) - New worker pool variables unlock the ability to have dedicated worker pools for different environments or tenant tags.

This is the [first release of six in 2020](/blog/2020-03/releases-and-lts/index.md) and we're excited to share it with the world. Keep reading to learn more about the updates. 

## Dark Mode

<iframe width="560" height="315" src="https://www.youtube.com/embed/TODO" frameborder="0" allowfullscreen></iframe>

Developers and operations folks love dark mode and I'm thrilled to share that Octopus now supports dark mode. It's a cool feature but it also easier on eyes, helps reduce eye strain and looks amazing. 😎

Octopus includes support to detect if your OS is running dark mode and can switch automatically. I highly recommend turning it on and giving it a test drive.

## Octopus and Octopus CLI are now more at home on Linux and macOS

Octopus aims to have world-class support for multiple platforms including Windows machines, Linux machines and popular cloud services. This release includes two changes to make Octopus feel more at home on Linux and macOS.

### Octopus CLI available via HomeBrew, APT and YUM 

![](octopus-cli-xplat.png)

The Octopus CLI, formerly known as `octo.exe`, has always been available via [Chocolately](https://chocolatey.org/packages/OctopusTools) which made it easy to install on Windows workstations. This same ease of installation is now available for macOS and Linux via HomeBrew, APT and YUM.

[Try it](https://octopus.com/downloads/octopuscli)

### Azure and AWS deployments work on Windows and Linux machines

TODO: Screenshot

## Worker Pools can be scoped by environment and tenants

TODO: Screenshot

This change is a small but significant as it enables teams to 

## What's next?

Check out our [public roadmap](https://octopus.com/roadmap) to see what's coming next. We're about to start working on some incredible new features including deep git integration unlocking pipeline as code.

## Conclusion

Octopus 2020.1 includes Dark Mode, cross platform Azure and AWS step templates, Octopus CLI on HomeBrew, APT and YUM package repositories, and new worker pool variables. This is a great start to the year and we're working hard to keep delivering updates.

Feel free to leave us a comment, and let us know what you think! Happy deployments!