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

With .NET Framework 4.5.2 reaching end of support from April 26, 2022, and .NET Core 3.1 reaching end of support from December 13, 2022, we're moving Tentacle to .NET Framework 4.8 for the Windows installer only, and .NET 6 for everything else including the Windows Docker image.

In this post, I explain our decision and answer some questions you might have regarding these version changes.

## Why are we moving Tentacle now?

Tentacle has been on .NET Framework 4.5.2 and .NET Core 3.1 for some time. We're moving these versions because .NET Framework 4.5.2 has reached end of support on April 26, 2022, and .NET Core 3.1 will reach end of support from December 13, 2022. After this, we'd miss out on security patches and library updates.

To continue providing our customers with secure software, we must align with the support schedules of our underlying development frameworks.

## What's the plan for Tentacle moving forward?

As a first step, we're moving Tentacle to .NET Framework 4.8 for the Windows installer, and .NET 6 for everything else. There are still many deployment targets using different versions of Windows 7 and 8 (including Windows 7 SP1 and Windows Server 2008 SP2), which need a supported .NET version for the short to medium term.

Eventually we'd like to drop support for .NET Framework and have everything running on .NET 6.

## Questions you might have

### Do I need to do anything about the .NET version change?

- If you're on Windows
  - Any version before Windows Server 2022 will need .NET Framework 4.8 runtime installed.
  - For Windows Server 2022 and later there's nothing to do.
- If you're on Linux/Mac
  - Make sure your OS is compatible with .NET 6.

When this version of Tentacle becomes available, you can upgrade your existing Tentacles using any of the methods we support.

To learn more about compatible .NET versions, please visit these Microsoft docs pages:

- [.NET Framework Compatibility](https://learn.microsoft.com/en-us/dotnet/framework/migration-guide/versions-and-dependencies#net-framework-48)
- [.NET Linux Compatibility](https://learn.microsoft.com/en-us/dotnet/core/install/linux)

### Will my deployments be impacted?

As long as your Tentacle OS meets the runtime requirements, your deployments won't be affected by this move. By including .NET Framework 4.8, we remain backward compatible as far back as Windows 7 SP1.

### Should I upgrade deployment targets on Windows 7 SP1 and Windows Server 2008 SP2?

Yes, we encourage you to upgrade your Tentacles that are still on Windows 7 SP1 or Windows Server 2008 SP2.

We recommend upgrading to a version that supports at least .NET Framework 4.8 and, ideally, a version compatible with .NET 6 and above.

### What if I can't upgrade my deployment targets?

If you can't upgrade to a supported .NET version, you need to lock your Tentacle version to avoid it automatically upgrading. Locking ensures your Tentacles remain functional.

To learn more about locking your Tentacle version, please read [our post about Tentacle versioning](https://octopus.com/blog/tentacle-versioning#lock-on-the-tentacle).

### When will the Windows Installer be upgraded to .NET 6?

We plan to move Tentacle to .NET 6 for the Windows installer in early to mid-2023. This gives you time to upgrade your deployment targets to a version that supports .NET 6.

## Conclusion

We hope this post explains why we're moving Tentacle to .NET Framework 4.8 and .NET 6.

If you have any questions or concerns, please get in touch with our [support team](mailto:support@octopus.com). 

Happy deployments!
