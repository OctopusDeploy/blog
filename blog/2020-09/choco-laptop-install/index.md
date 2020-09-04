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

I recently picked up a new Dell XPS 15" laptop after being on a Macbook Pro for the last 2 years. I looked back at Bob Walkers [Automating developer machine setup with Chocolatey](https://octopus.com/blog/automate-developer-machine-setup-with-chocolatey) blog, and it got me thinking about how I could automate it with [Chocolatey](chocolatey.org/) and [Runbooks](https://octopus.com/docs/runbooks). I would recommend spending time going through Bob's blog before or after reading this blog as it may help if you're new to Chocolatey.

I use Chocolatey a lot when provisioning Cloud and On-Premises servers and, in the past when I was working in Operations and using Infrastructure as Code I used Chocolatey to install a range of tools on Servers, Laptops and Desktops. I recently presented a Webinar with Paul Broadwith from Chocolatey about [Operations Automation with Octopus Runbooks and Chocolatey](https://www.youtube.com/watch?v=E0z4QbwTuBg) which displayed how easy this was to do with Runbooks on Cloud infrastructure. In the webinar, we focus on using ARM Templates, Octopus Runbooks and Chocolatey to provision Developer machines and Server on [Azure](https://azure.microsoft.com/en-gb/overview/what-is-azure/).

The problem I am trying to solve here, is to automate all of my packages in a single Octopus Runbook using Chocolatey to install every application I use in my day to day role. Also, everyone loves a fresh install of Windows, and I am no different. There's something fresh about a brand new install, and I may look at refreshing my laptop every so often, and I wanted a way to automate that and make it as painless as possible. I'd also like to share this with other members of my team so they can use this script whenever they want to re-prep their windows laptop or even a server.

!toc

## What is Chocolatey

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

## Why use Runbooks and Chocolatey

[Runbooks](https://octopus.com/docs/runbooks) is my favourite feature of Octopus Deploy. I think it's due to my Operations background and I can see how it automate all those tasks that take up so much time in Operations that will allow Admins, DevOps Engineers and similar to automate the mundane away from your day. Being in Operations can be a hard position to be. It can be a little thankless at times, and it reminds me of times where people wondering what you're doing as everything "just works", and then they come to you and complain what we're even doing "when everything breaks".

Let's be honest, how many times can you install [IIS](https://www.iis.net/) or [SQL](https://en.wikipedia.org/wiki/Microsoft_SQL_Server) before it becomes boring, repetitive and error prone?

You can use [Runbooks](https://octopus.com/docs/runbooks) to automate this part away to allow you to focus on more interesting problems:

- [Routine Operations](https://octopus.com/docs/runbooks/runbook-examples/routine)
- [Emergency Operations](https://octopus.com/docs/runbooks/runbook-examples/emergency)
- [Database Operations](https://octopus.com/docs/runbooks/runbook-examples/databases)
- [AWS Operations](https://octopus.com/docs/runbooks/runbook-examples/aws)
- [Azure Operations](https://octopus.com/docs/runbooks/runbook-examples/azure)
- [GCP Operations](https://octopus.com/docs/runbooks/runbook-examples/gcp)
- [Terraform](https://octopus.com/docs/runbooks/runbook-examples/terraform)

## Let's get going

This blog will not deal with installing the Operating System on your developer machine. There are loads of great tools out there that you can use for load the latest Windows Desktop and Server Operating Systems, and I am going to assume you've added the [Octopus Tentacle](https://octopus.com/docs/infrastructure/deployment-targets/windows-targets) as part of this process. Some tools I've used in the past when prepping Servers, Laptops and Desktops are:

- [Microsoft System Centre Configuration Manager](https://en.wikipedia.org/wiki/Microsoft_System_Center_Configuration_Manager)
- [Windows Deployment Services](https://docs.microsoft.com/en-us/windows/deployment/windows-deployment-scenarios-and-tools)
- [Packer](https://www.packer.io/)

Generally, public cloud providers also provide tools to allow you to do Infrastructure as Code such as:

- [Azure Resource Manager Templates](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/overview)
- [AWS Cloudformation](https://aws.amazon.com/cloudformation/)
- [Google Deployment Manager](https://cloud.google.com/deployment-manager/docs)
- [Terraform](https://www.terraform.io/)

## Preparation

The first thing I did here, was setup a brand new [Space](https://octopus.com/docs/administration/spaces) in Octopus called **LaptopPrep**. As part of the new Space configuration I created:

- A single Octopus [environment](https://octopus.com/docs/infrastructure/environments) named **Prep**

![](images/environment.png "width=500")

- Added my laptop as a [deployment target](https://octopus.com/docs/infrastructure/deployment-targets) and assigned it the [target role](https://octopus.com/docs/infrastructure/deployment-targets#target-roles) or **Laptop**.

![](images/deploymenttarget.png "width=500")

- Checked the Infrastructure tab had my new laptop as healthy.

![](images/healthy.png "width=500")

- Last thing I had to do, was create a [Project](https://octopus.com/docs/projects) called **Laptop Provisioning**

![](images/project.png "width=500")

## Runbook Configuration

I want to do more than just install Chocolatey packages and I'll touch on these briefly as part of this blog.

First thing you will want to do is browse to the Project, and add the Runbook named **Install Laptop Dependencies**

### Set Timezone, Input & Region

One thing that bugs me about setting up Windows, is having to configure my non-default regions which isn't the US. So, as part of this I use a PowerShell script to set this for all my laptops, desktops and so forth. The below works for me, and you can tweak it to your requirements.

```PowerShell
#Set home location to United Kingdom
Set-WinHomeLocation 0xf2

#override language list with just English GB
$1 = New-WinUserLanguageList en-GB
$1[0].Handwriting = 1
Set-WinUserLanguageList $1 -force

#Set system local
Set-WinSystemLocale en-GB

#Set the timezone
Set-TimeZone "GMT Standard Time"
```

![](images/addrunbook.png "width=500")

### Checking if Chocolatey is installed

The next step I used, was a community contributed step template called [Chocolatey - Ensure Installed](https://library.octopus.com/step-templates/c364b0a5-a0b7-48f8-a1a4-35e9f54a82d3/actiontemplate-chocolatey-ensure-installed). This step has a single purpose, and it's to check if Chocolatey is installed, and to install if it's not.

![](images/chocolateyinstallstep.png "width=500")

### Install Chocolatey Package Step

If you do browse the webinar, you will see we had different steps for different packages. Paul Broadwith has updated our Chocolatey community step template that installs all of your chocolatey packages in a single step and you can see it on our [Community Step Template Library](https://library.octopus.com/step-templates/b2385b12-e5b5-440f-bed8-6598c29b2528/actiontemplate-chocolatey-install-package)

I made a list of all of the packages I needed to install on my new laptop, and not all of this you will find useful as I do some webinars, and I can often be found in [Camtasia](https://www.techsmith.com/video-editor.html), [OBS](https://obsproject.com/) as much as I can be found in [VSCode](https://code.visualstudio.com/), SQL and so forth.

The applications I found I needed on my new laptop is below:

```
git vscode sql-server-management-studio slack sql-server-2019 github-desktop obs-studio rdmfree googlechrome firefox spotify octopusdeploy octopusdeploy.tentacle dotnet4.7.2 dotnetfx dotnetcore 7zip visualstudio2019professional nordvpn lastpass-chrome lastpass docker-desktop chromium googledrive google-drive-file-stream helm kubernetes-cli minikube zoom streamdeck notepadplusplus nugetpackageexplorer sdio virtualbox jre8 vlc python foxitreader putty.install sysinternals camtasia snagit vagrant packer terraform vmwareworkstation
```

![](images/chocoinstall.png "width=500")

#### Install Chocolatey Package step parameters

As you've saw, there are a few paramters which can be set, and I wanted to explain these so you get the full picture.

In the **(Optional) Version** Parameter selection, you can specify a specific version of the software you want to install. If you're using more than one package per step, and want to specify specific versions of software, you will need to configure that Chocolatey Install and add the version in an additional step.

In the **(Optional) Cache Location** Parameter selection, you can specify a non default location for the cache. I found this useful when installing SQL without having the tentacle run as an administrator. I found that SQL can be a little tricky to install without running the Tentacle Service as a local administrator and you can specify a folder such as **C:\Octopus\Applications** as the cache which the Local System User has full access to.

The **(Optional) Package source** Parameter is probably the single most important parameter in this step. If you're doing this at home, then you might be fine to use the Chocolatey Package Repository which is the default setting. However, if you're doing this for a company, please consider using your own package source repository such as [Nexus](https://www.sonatype.com/nexus/repository-pro), [Artifactory](https://jfrog.com/artifactory/) or [MyGet](https://www.myget.org/).

The Chocolatey package resource is built by the community, for the community and if you're using the community respository for Enterprise or large scale package installation, you are likely to be [rate limited](https://chocolatey.org/docs/community-packages-disclaimer#rate-limiting). Be careful, and be kind to the community.

The last two options cover whether you want to see the download progress in your logs, and generally I turn them off as you can land up with hundreds of thousands of log files. The other is to allow for additional parameters.

![](images/chocoparams.png "width=500")

## Running the Runbook


