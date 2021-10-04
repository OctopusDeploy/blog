---
title: Raising the minimum requirements for hosting and using Octopus Server
description: In 2020, we will raise the minimum requirements for hosting and using Octopus Server.
author: michael.noonan@octopus.com
visibility: public
published: 2019-11-27
metaImage: minimum_requirements_octopus_2020.png
bannerImage: minimum_requirements_octopus_2020.png
bannerImageAlt: Raising the minimum requirements for hosting and using Octopus Server
tags:
- Product
---

![Raising the minimum requirements for hosting and using Octopus Server](minimum_requirements_octopus_2020.png)

:::success
**September 2020 Update**

We have listened to our customers and have revised our SQL Server requirement to SQL Server 2016+, down from SQL Server 2017+

This has taken affect from the following versions:
- 2020.1.x *(SQL Version change was not enforced in 2020.1.x)*
- 2020.2.18 ➜ 2020.2.latest
- 2020.3.6 ➜ 2020.3.latest
- 2020.4.0 ➜ latest

Any versions that are in the following version ranges still have the SQL Server 2017+ version requirement enforced
- 2020.2.0 ➜ 2020.2.17 
- 2020.3.0 ➜ 2020.3.5
:::

## Original post

In 2020, we will raise the minimum requirements for hosting and using Octopus Server:

- Windows Server 2012 R2
- SQL Server 2017+

Additionally, we're also ending mainstream support for IE11.

Supporting older servers and browsers drains our time and attention, making it harder for us to innovate and move the Octopus ecosystem forward. We want to break this cycle in 2020. From another perspective, if Octopus Server is a critical resource in your business, you really should consider a modern operating environment for improved security and performance.

Maybe this is the kind of push we all needed? Maybe we haven’t fully considered how difficult this will make your life? If these minimum requirements will be a problem in your scenario, please tell us why in the comments, or by [reaching out to us](https://octopus.com/support). The rest of this blog post should answer the most common questions.

Happy Deployments!

## Question: Will I be affected?

You will only be affected if:

1. You host Octopus Server on Windows Server 2008-2012 or SQL Server 2008-2014 and you want to upgrade to Octopus Server `2020.x`.
2. You use Internet Explorer 11, by force or your own preference, regardless of whether you host Octopus Server on your own infrastructure or use Octopus Cloud.

## Question: Will my deployments be affected?

No, your deployments will not be affected. These requirements only affect the hosting of Octopus Server itself. We remain highly-backwards compatible for your deployment targets in Octopus including older Windows and Linux operating systems.

## Question: How much time do I have to plan my upgrade?

We will ship one more LTS release of Octopus Server in January 2020, with the version `2019.12 LTS`, which will be supported under our [long-term support program](https://octopus.com/docs/administration/upgrading/long-term-support) **until July 2020**, after which you should seriously consider upgrading Octopus Server. We hope this gives you sufficient time to plan the full upgrade.

## Question: Should I upgrade Octopus Server anyhow?

Staying up to date helps on both sides of this relationship:

- We can give you the best kind of support when you run modern Octopus Server. We can turn around issues with less overhead, and upgrading is less risky when you apply incremental upgrades.
- You benefit from everything we’ve done since you last upgraded, including delightful new features like the streamlined process editor and Operations Runbooks. Take a look at [what’s new](https://octopus.com/whatsnew). Also, [take a look at our dynamic roadmap](https://octopus.com/roadmap) to see what we are planning to ship very soon.
- We are proactive when it comes to security and trust. Keeping your Octopus Server up to date is the foundation of [a secure and trustworthy installation](https://octopus.com/docs/administration/security/hardening-octopus). Without wanting to be alarmist, here is a list of [reasons to upgrade from a security perspective](https://www.cvedetails.com/vulnerability-list/vendor_id-16785/product_id-39115/Octopus-Octopus-Deploy.html).

## Question: Why will Windows Server 2012 R2 be the minimum operating system?

The driving reason for us is simple: [.NET Core 3.x requires Window Server 2012 R2](https://github.com/dotnet/core/blob/master/release-notes/3.0/3.0-supported-os.md), and we want to have a single target for Octopus Server. Right now, we need to target the full .NET framework for Windows and .NET Core 2.x for Linux. Targeting .NET Core 3.x will simplify our codebase and deployment pipeline. Also, we prefer to offer world-class support for a single framework across a wider variety of Windows and Linux host environments.

Additionally, mainstream support ended for Windows Server 2012 back in October 2018. Octopus is a critical part of your infrastructure, but it is only as trustworthy and secure as the underlying operating system. Upgrading is just a good idea.

## Question: How do I upgrade my host operating system?

You can perform an in-place upgrade of your Windows Server operating system to a more modern operating system. Learn about [upgrading Windows Server](https://docs.microsoft.com/en-us/windows-server/get-started/installation-and-upgrade).

Alternatively, you can move your Octopus Server to another host which is already running a more modern operating system. Learn about [moving your Octopus Server](https://octopus.com/docs/administration/managing-infrastructure/moving-your-octopus).

## Question: Why will SQL Server 2017 be the minimum database?

The driving reason for us is simple: We make heavy use of JSON in our hybrid data store, and we want to leverage all the JSON features offered by SQL Server 2017. In the past, we’ve hesitated pushing this agenda because we don’t want to burden our customers. Reflecting on the last few months there are many scenarios where a modern database foundation would have helped us offer more value:

1. We could have helped customers solve complex support problems using first-class JSON queries rather than using brittle SQL queries.
2. We could have authored more robust database upgrade scripts for the same reason.
3. We could have authored more efficient database queries making Octopus faster, rather than being constrained by the lowest-common denominator.
4. We could have leveraged new database engine features to make every Octopus installation faster.

We don’t see any value progressively rolling from SQL Server 2012 to 2014 to 2016, forcing you to upgrade each time. We think it is time to rip off the proverbial Band-Aid(TM), and jump straight to SQL Server 2017, so we can offer better value and faster turnaround to all of our customers.

## Question: How do I upgrade my database server?

You can perform an in-place upgrade of your SQL Server. Learn about [upgrading to SQL Server 2017](https://docs.microsoft.com/en-us/sql/database-engine/install-windows/upgrade-sql-server).

## Question: Octopus is using a shared database server which will be hard to upgrade! What about us?

We don’t want upgrading Octopus Server to be hard, hopefully that hasn’t been your experience in the last 5 years! However, we believe the end result will be worth the effort. In this situation, all of the applications hosted on that database server will probably benefit from the improvements to the SQL Server engine. [Here is a good post on the subject: Reasons to Upgrade to SQL Server 2017](https://sqlperformance.com/2018/04/sql-server-2017/reasons-to-upgrade-to-sql-server-2017).

Alternatively, you can consider moving your Octopus to a free edition of SQL Server 2017 until the shared database server is upgraded.

Another alternative is to consider [moving to Octopus Cloud](https://octopus.com/cloud) where we take care of everything on your behalf.

If you will be really stuck, please [reach out for a chat](https://octopus.com/support)!

## Question: What about Internet Explorer 11?

We aren’t going to prevent IE11 from accessing Octopus. We will also leave all the shims in place to keep IE11 working as-is with Octopus for a while longer. However, we will not actively target IE11 during feature development and testing, and we will not actively fix bugs which only affect IE11. Yes, this will probably lead to a degraded experience using IE11 with Octopus over time.

From a selfish point of view, only 3% of our traffic comes from IE nowadays, and the time and effort we spend catering to IE11 does not feel worthwhile.

From a selfless point of view, we don’t want you using IE11 when you really should be using a modern browser instead. Even the Microsoft team are [pushing this agenda in 2019](https://techcommunity.microsoft.com/t5/Windows-IT-Pro-Blog/The-perils-of-using-Internet-Explorer-as-your-default-browser/ba-p/331732). IE11 is a compatibility solution for web apps which cannot work with other browsers, but this is not Octopus.

At some point in the future we will remove all the IE11-specific code and prevent anything other than modern browsers from using Octopus resulting in a better future for all of us.

## Question: When can I host Octopus Server on Linux?

In the near future. We are already doing this ourselves internally and for Octopus Cloud (our SaaS platform). We will post a separate announcement when we are ready to support Octopus Server installations on your own Linux infrastructure.
