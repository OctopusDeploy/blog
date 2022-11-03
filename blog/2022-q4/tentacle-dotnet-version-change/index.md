---
title: "Tentacle .NET Version Change"
description: Moving Tentacle from .NET Framework 4.5.2 & .NET Core 3.1 to .NET Framework 4.8 & .NET 6
author: ian.khor@octopus.com
visibility: public
published: 2022-11-07-1400
metaImage: ../q4-template-folder/placeholderimg.png
bannerImage: ../q4-template-folder/placeholderimg.png
bannerImageAlt: Moving Tentacle from .NET Framework 4.5.2 & .NET Core 3.1 to .NET Framework 4.8 & .NET 6.
isFeatured: false
tags:
  - Product
---

We are moving Tentacle to `.NET Framework 4.8` for the Windows installer, and `.NET 6` for everything else including the Windows Docker image, in the coming months.

## The Change

Tentacle had been on `.NET Framework 4.5.2` and `.NET Core 3.1` for some time. The decision to move these versions was taken in light of `.NET Framework 4.5.2` being EOL from 26 April 2022 and `.NET Core 3.1` being EOL from 13 December 2022, which means we'll be missing out on security patches and library updates. In order to ensure we can continue to provide our customers with software that is safe and secure, it is essential that we align with the support schedules of our underlying development frameworks.

## The Plan

We are going to compile Tentacle for `.NET 6` as well as `.NET Framework 4.8` rather than only producing builds for `.NET 6`, as there are still a substantial number of deployment targets still using different versions of Windows 7 and 8 - including Windows 7 SP1 and Windows Server 2008 SP2 - which continue to require a supported .NET Framework version in the short to medium term and moving to `.NET 6` would cut off support for these servers. The `.NET 6` builds of Octopus Tentacle will be used for all other platforms, such as Linux packages and Docker Images.

The rest of this blog post should answer some common questions. As always, please reach out to our support team if you have any questions or concerns. Happy Deployments!

## FAQ

### Do I need to do anything to effect the .NET version change?

You would need to do the following if it applies to you:

- If not already, install `.NET Framework 4.8` on your Windows server operating system.
  - If you are on Windows Server 2022, please ignore this step as you will already have `.NET Framework 4.8` installed. Click [here](https://learn.microsoft.com/en-us/dotnet/framework/migration-guide/versions-and-dependencies#net-framework-48) to learn more.
- Upgrade your existing Tentacle via the usual methods we support or by using the 2023.1 bundled Tentacle when it becomes available.

### Will my deployments be affected?

No, your deployments will not be affected by this move. By making the incremental move to `.NET Framework 4.8` and `.NET 6`, we remain highly backward compatible for your deployment targets as far back as Windows 7 SP1.

### Should we upgrade deployment targets on Windows 7 SP1 and Windows Server 2008 SP2?

Yes, we encourage you to upgrade your deployment targets that are still on either Windows 7 SP1 or Windows Server 2008 SP2 to a version that supports `.NET Framework 4.8`, ideally a version that is compatible with `.NET 6` and above.

Where you are unable to upgrade your deployment targets to a supported .NET version, you’ll need to lock your Tentacle version to ensure it remains functional for those particular deployment targets. To learn more about locking your Tentacle version, please read this [blog post](https://octopus.com/blog/tentacle-versioning) here.

To learn more about compatible framework versions, please visit this [link from Microsoft](https://learn.microsoft.com/en-us/dotnet/framework/migration-guide/versions-and-dependencies#net-framework-48).

### When will the Windows Installer be upgraded to `.NET 6`?

As an initial indication, we are likely to move Tentacle to `.NET 6` for the Windows installer either early or mid 2023 to give you time to upgrade your deployment targets to a version that supports `.NET 6` moving forward.
