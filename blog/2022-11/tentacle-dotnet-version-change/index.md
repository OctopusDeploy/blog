---
title: "Tentacle .NET version change"
description: Learn why we're moving Tentacle from .NET Framework 4.5.2 & .NET Core 3.1 to .NET Framework 4.8 & .NET 6.
author: ian.khor@octopus.com
visibility: public
published: 2022-11-16-1400
metaImage: blogimage-tentacle-2022.png
bannerImage: blogimage-tentacle-2022.png
bannerImageAlt: Octopus tentacle wrapped around the .NET logo in front of a server
isFeatured: false
tags:
  - Product
---

With .NET Framework 4.5.2 being end of life (EOL) from April 26, 2022, and .NET Core 3.1 being EOL from December 13, 2022, we're moving Tentacle to .NET Framework 4.8 for the Windows installer only, and .NET 6 for everything else including the Windows Docker image.

In this post, I explain our decision and answer some questions you might have regarding the Tentacle bump.

## Why are we moving Tentacle now?

Tentacle has been on .NET Framework 4.5.2 and .NET Core 3.1 for some time. We're moving these versions because .NET Framework 4.5.2 will be EOL from April 26, 2022, and .NET Core 3.1 will be EOL from December 13, 2022. After this, we'd miss out on security patches and library updates. 

To continue providing our customers with secure software, we must align with the support schedules of our underlying development frameworks.

## What's the plan for Tentacle moving forward?

We'll compile Tentacle for .NET 6 and .NET Framework 4.8, as there are still many deployment targets using different versions of Windows 7 and 8 (including Windows 7 SP1 and Windows Server 2008 SP2), which need a supported .NET version for the short to medium term.

The .NET 6 builds of Octopus Tentacle will be used for all other platforms, such as Linux packages and Docker images.

## Questions you might have

### Do I need to do anything to support the .NET version change?

You need to do the following if it applies to you:

- If you haven't done so, install .NET Framework 4.8 on your Windows server operating system.
  - If you're on Windows Server 2022, you can ignore this step as you'll already have .NET Framework 4.8 installed. [Learn more in the Microsoft docs](https://learn.microsoft.com/en-us/dotnet/framework/migration-guide/versions-and-dependencies#net-framework-48).
- Upgrade your existing Tentacle using the usual methods we support, or use the version of Tentacle bundled with Octopus 2023.1 when it becomes available.

### Will my deployments be impacted?

No, your deployments won't be affected by this move. By making the incremental move to .NET Framework 4.8 and .NET 6, we remain backward compatible with your deployment targets as far back as Windows 7 SP1.

### Should I upgrade deployment targets on Windows 7 SP1 and Windows Server 2008 SP2?

Yes, we encourage you to upgrade your deployment targets that are still on Windows 7 SP1 or Windows Server 2008 SP2. 

We recommend upgrading to a version that supports .NET Framework 4.8 and, ideally, a version compatible with .NET 6 and above.

### What if I can't upgrade my deployment targets?

If you can't upgrade your deployment targets to a supported .NET version, you need to lock your Tentacle version to avoid it automatically upgrading. Locking ensures it remains functional for those deployment targets. 

To learn more about locking your Tentacle version, please read [our post about Tentacle versioning](https://octopus.com/blog/tentacle-versioning#lock-on-the-tentacle).

To learn more about compatible .NET versions, please visit these Microsoft docs pages:

- [.NET Framework Compatibility](https://learn.microsoft.com/en-us/dotnet/framework/migration-guide/versions-and-dependencies#net-framework-48)
- [.NET Linux Compatibility](https://learn.microsoft.com/en-us/dotnet/core/install/linux)

### When will the Windows Installer be upgraded to .NET 6?

We plan to move Tentacle to .NET 6 for the Windows installer in early to mid-2023. This gives you time to upgrade your deployment targets to a version that supports .NET 6.

## Conclusion

We hope this post explains why we're moving Tentacle to .NET Framework 4.8 for the Windows installer, and .NET 6 for everything else including the Windows Docker image. 

If you have any questions or concerns, please get in touch with our [support team](mailto:support@octopus.com). 

Happy deployments!
