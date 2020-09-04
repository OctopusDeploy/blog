---
title: Configuring a developer machine with Chocolatey and Octopus Runbooks
description: Learn how to automate your development machine setup with Chocolatey and Octopus Runbooks
author: derek.campbell@octopus.com
visibility: public
published: 2020-09-30
metaImage: 
bannerImage: 
tags:
 - Product
 - Runbooks
 - Chocolatey
---

I recently picked up a new Dell XPS 15" laptop after being on a Macbook Pro for the last 2 years. I looked back at Bob Walkers [Automating developer machine setup with Chocolatey](https://octopus.com/blog/automate-developer-machine-setup-with-chocolatey) blog, and it got me thinking about how I could automate it with [Chocolatey](chocolatey.org/) and [Runbooks](https://octopus.com/docs/runbooks).

I use Chocolatey a lot when provisioning Cloud and On-Premises servers and, in the past when I was working in Operations and using Infrastructure as Code I used Chocolatey to install a range of tools on Servers, Laptops and Desktops. I recently presented a Webinar with Paul Broadwith from Chocolatey about [Operations Automation with Octopus Runbooks and Chocolatey](https://www.youtube.com/watch?v=E0z4QbwTuBg) which displayed how easy this was to do with Runbooks on Cloud infrastructure.

!toc

## What is Chocolatey?

Chocolatey is a Package Manager for Windows. It's an open source project that provides Developers, Operations and everything in between a way to manage, install and upgrade software across their Windows estate. Chocolatey is focussed on helping making managing Windows software easier, more streamlined and accessible to everyone using a Windows computer. If you want to find out more about installing Chocolatey without Runbooks then check out the Chocolatey Install [doc](https://chocolatey.org/install).

You don't need Runbooks to use Chocolatey, and it's as simple as opening a Windows PowerShell window and running a script to install something like google chrome:

```PowerShell
choco install googlechrome -y
```

If you wanted to install more than a single application, you could write some PowerShell scripts and execute them locally such as:

```PowerShell
Write-Host "Installing SQL Server Management Studio"
choco install sql-server-management-studio -y

Write-Host "Installing SQL Server Developer Edition"
choco install sql-server-2019 -y

Write-Host "Installing Github Desktop"
choco install github-desktop -y
```

You could extend this out to all of your required applications, and source control the script to somewhere like [Github](Github.com) and allow read access, so the script can be run by users or during machine provisioning and this will automate almost all of your application installation.

### Chocolatey Packages

Chocolatey is an open source tool and you can get lots of pre-configured packages from the site. In my experience though, most organizations write their own packages, and you can do this too and you can see more about this on the [Create Chocolatey Package Page](https://chocolatey.org/blog/create-chocolatey-packages). The main reason for writing your own package is because:

- Company purchased licenses that need to be contained in the package.
- Custom configuration such as a backup agent which needs to replicate to SiteA from SiteB.
- Community package may not exist.

If you are writing your own package, consider sharing it to the Chocolatey community and you can read more about that on the [Chocolatey site](https://chocolatey.org/docs/create-packages).

You can install Octopus from a chocolatey package, and we publish each new version as soon as it's available which

## Why use Runbooks?

[Runbooks](https://octopus.com/docs/runbooks) is my favourite feature of Octopus Deploy. I think it's due to my Operations background and I can see how it automate all those tasks that take up so much time in Operations that will allow Admins, DevOps Engineers and similar to automate the mundane away from your day. Being in Operations can be a hard position to be. It can be a little thankless at times, and it reminds me of times where people wondering what you're doing as everything "just works", and then they come to you and complain what we're even doing "when everything breaks".

Let's be honest, how many times can you install [IIS](https://www.iis.net/) or [SQL](https://en.wikipedia.org/wiki/Microsoft_SQL_Server) before it becomes boring, repetitive and error prone? You can use [Runbooks](https://octopus.com/docs/runbooks) to automate this part away to allow you to focus on more interesting problems:

- [Routine Operations](https://octopus.com/docs/runbooks/runbook-examples/routine)
- [Emergency Operations](https://octopus.com/docs/runbooks/runbook-examples/emergency)
- [Database Operations](https://octopus.com/docs/runbooks/runbook-examples/databases)
- [AWS Operations](https://octopus.com/docs/runbooks/runbook-examples/aws)
- [Azure Operations](https://octopus.com/docs/runbooks/runbook-examples/azure)
- [GCP Operations](https://octopus.com/docs/runbooks/runbook-examples/gcp)
- [Terraform](https://octopus.com/docs/runbooks/runbook-examples/terraform)
