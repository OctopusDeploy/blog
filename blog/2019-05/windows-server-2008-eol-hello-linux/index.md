---
title: Ending Support for Octopus Server on Windows Server 2008. Introducing Octopus Server on Linux!
description: Octopus Server 2019.3 LTS will be the final version of Octopus Server which can be hosted on Windows Server 2008, paving the way for Octopus Server on Linux!
author: michael.noonan@octopus.com
visibility: public
published: 2019-05-21
metaImage: blogimage-octopusserverlinux-2.png
bannerImage: blogimage-octopusserverlinux-2.png
tags: 
- New Releases
---

**Octopus Server 2019.3 LTS will be the final version of Octopus Server you can host on Windows Server 2008.**

This decision paves the way for a truly cross-platform Octopus Server. In the near future you will be able to host Octopus Server on any modern Windows or Linux operating system, or a container on either platform.  To make this happen we need to move our minimum requirements for hosting Octopus Server up to **Windows Server 2008 R2**.

It's also worth noting Microsoft is [ending extended support for Windows Server 2008 and 2008 R2 in January 2020](https://docs.microsoft.com/en-us/windows-server/get-started/installation-and-upgrade) and thankfully you can [perform an in-place upgrade](https://docs.microsoft.com/en-us/windows-server/get-started/installation-and-upgrade). If Octopus Server is a critical resource in your business, you really should consider hosting it on a modern operating system for improved security and performance.

The rest of this blog post should answer the most common questions. As always, please reach out to [our support team](https://octopus.com/support) if you have any questions or concerns!
Happy Deployments!


## Question: Will I be affected?

You will only be affected if:

1. You host Octopus Server on Windows Server 2008, and
2. You want to upgrade Octopus Server beyond `2019.3 LTS`.

Don't worry, the Octopus Server installer will prevent you from accidentally upgrading. If you do want to upgrade Octopus Server beyond `2019.3 LTS` you will need to upgrade your host operating system.

## Question: Can we still host Octopus Server on Windows Server 2008 R2?

Yes, Windows Server 2008 R2 will stil be a supported host for Octopus Server.

There is no practical reason for us to exclude Windows Server 2008 R2 right now, even though Microsoft is ending extended support for Windows Server 2008 R2 in January 2020.

## Question: Will my deployments be affected?

No, your deployments will not be affected by upgrading Octopus Server beyond `2019.3 LTS`. This only affects the hosting of Octopus Server itself. We remain highly-backwards compatible for your deployment targets in Octopus, even as far back as `Windows Server 2003`.

## Question: How do I upgrade my host operating system?

You can perform an in-place upgrade of your Windows Server 2008 operating system to a more modern operating system. Learn about [upgrading Windows Server](https://docs.microsoft.com/en-us/windows-server/get-started/installation-and-upgrade#upgrading-from-windows-server-2008-r2-or-windows-server-2008).

Alternatively, you can move your Octopus Server to another host which is already running a more modern operating system. Learn about [moving your Octopus Server](https://octopus.com/docs/administration/managing-infrastructure/moving-your-octopus).

## Question: How long will you support Octopus Server 2019.3 LTS and Windows Server 2008?

We will continue to support Octopus Server `2019.3 LTS` under our [long-term support program](https://octopus.com/docs/administration/upgrading/long-term-support) **until October 2019**. To upgrade Octopus Server beyond `2019.3 LTS` you will need to upgrade your host operating system to **Windows Server 2008 R2 or newer**.

## Question: When can I host Octopus Server on Linux?

In the near future. We are already doing this ourselves internally. Octopus Server `2019.5`, our current release in the fast lane, will start to introduce cross-platform capabilities. After this we will focus on the stability and performance of Octopus Server on Linux, followed by a fully supported release of Octopus Server running on Linux.