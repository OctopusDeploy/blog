---
title: Sunset support for hosting Octopus Server on Windows Server 2008
description: Octopus Server 2019.3 LTS will be the final version of Octopus Server which can be hosted on Windows Server 2008. We are doing this to pave the way for a truly cross-platform Octopus Server which can be hosted on any modern Windows or Linux operating system, or in a container of your choice on those platforms.
author: michael.noonan@octopus.com
visibility: private
published: 2019-05-20
metaImage: TODO.png
bannerImage: TODO.png
tags:
---

**Octopus Server 2019.3 LTS will be the final version of Octopus Server you can host on Windows Server 2008.**

We are doing this for a few reasons:

- It paves the way for a truly cross-platform Octopus Server. You will be able to host Octopus Server on any modern Windows or Linux operating system, or a container on either platform. To make this happen we need to move our **minimum requirements for hosting Octopus Server to Windows Server 2008 R2**. Watch this space: Octopus Server on Linux is coming soon!
- Microsoft is ending extended support for Windows Server 2008 in 2020.
- Octopus Server is a critical resource in your business and you should consider hosting it on a modern operating system for improved security and performance.

## Question: Will I be affected?

You will only be affected if:

1. You host Octopus Server on Windows Server 2008, and
2. You want to upgrade Octopus Server beyond `2019.3 LTS`.

Don't worry, the Octopus Server installer will prevent you from accidentally upgrading. If you do want to upgrade Octopus Server beyond `2019.3 LTS` you will need to upgrade your host operating system.

## Question: Will my deployments be affected?

No, your deployments will not be affected by upgrading Octopus Server beyond `2019.3 LTS`. This only affects the hosting of Octopus Server itself. We remain highly-backward compatible for your deployment targets in Octopus, even as far back as `Windows Server 2003`.

## Question: How do I upgrade my host operating system?

You can perform an in-place upgrade of your Windows Server 2008 operating system to a more modern operating system. Learn about [upgrading Windows Server](https://docs.microsoft.com/en-us/windows-server/get-started/installation-and-upgrade#upgrading-from-windows-server-2008-r2-or-windows-server-2008).

You can also move your Octopus Server to another host which is running a more modern operating system. Learn about [moving your Octopus Server](https://octopus.com/docs/administration/managing-infrastructure/moving-your-octopus).

## Question: How long will you support Octopus Server 2019.3 LTS and Windows Server 2008?

We will continue to support Octopus Server `2019.3 LTS` under our [long-term support program](https://octopus.com/docs/administration/upgrading/long-term-support) **until October 2019**. To upgrade Octopus Server beyond `2019.3 LTS` you will need to upgrade your host operating system to **Windows Server 2008 R2 or newer**.