---
title: Configure Windows Servers with Chocolatey, PowerShell, and Octopus Runbooks
description: This blog post shows you how to automate your Windows servers setup with Chocolatey, PowerShell, and Octopus Runbooks
author: derek.campbell@octopus.com
visibility: private
published: 3000-01-01
metaImage: automate_machine_chocolately.png
bannerImage: automate_machine_chocolately.png
bannerImageAlt: Configure team member’s machines with Chocolatey, PowerShell, and Octopus Runbooks
tags:
 - Product
 - Runbooks
 - Chocolatey
---

![Configure windows servers with Chocolatey, PowerShell, and Octopus Runbooks](automate_machine_chocolately.png)

Runbooks are a way to automate routine tasks that are commonly performed. One such task is the setup and installation of windows servers. In this blog, I demonstrate how to set up and install developer dependencies on a windows server using a Octopus Deploy runbook. The runbook can be used to set up any number of windows machines.

!toc

## What is Chocolatey

[Chocolatey](https://chocolatey.org/) is a package manager for Windows. It’s an open-source project that provides developers, operations, and everybody in between a way to manage, install, and upgrade software across their Windows estate. Chocolatey focuses on making managing Windows software more straightforward, streamlined, and accessible to everyone using a Windows computer. If you want to find out more about installing Chocolatey without runbooks, check out the Chocolatey Install [doc](https://chocolatey.org/install).

### Using Chocolatey

You don’t need Runbooks to use Chocolatey, and it’s as simple as opening an Administrator Windows PowerShell window and running a script to install something like google chrome:

```PowerShell
choco install googlechrome -y
```

If you wanted to install more than a single application, you could write PowerShell scripts and execute them locally:

```PowerShell
Write-Host "Installing Chocolatey Apps"
choco install sql-server-management-studio sql-server-2019 github-desktop git firefox -y
```

You could extend this out to all of your required applications, and source control the script somewhere with read access so the script can be run by users or during machine provisioning. This would automate almost all of your application installation.

### Chocolatey Packages

Chocolatey is an open-source tool, and you can get lots of pre-configured packages from the site. In my experience, though, most organizations write their own packages, and you can do this too. You learn more about this on the [Create Chocolatey Package page](https://chocolatey.org/blog/create-chocolatey-packages). The main reasons for writing your own package are:

- Company purchased licenses that need to be contained in the package.
- Custom configuration, such as a backup agent that needs to replicate to SiteA from SiteB.
- Community package may not exist.

If you are writing your own package, consider sharing it with the Chocolatey community. You can read more about that on the [Chocolatey site](https://chocolatey.org/docs/create-packages).

You can install Octopus from a Chocolatey package. We publish each new version as soon as it’s available, which happens automatically from our [TeamCity](https://www.jetbrains.com/teamcity/) build server once it’s available on our website. Read more about the [Octopus Deploy Chocolatey Package](https://chocolatey.org/packages/OctopusDeploy).

To install Octopus Deploy as a Chocolatey package, you can run the following:

```PowerShell
choco install OctopusDeploy -y
```

:::hint
You still need to configure Octopus after using Chocolatey to install, you can automate this, and there‘s more information on [Automating Octopus Installation](https://octopus.com/docs/installation/automating-installation).
:::

## Why use runbooks and Chocolatey

[Runbooks](https://octopus.com/docs/runbooks) is my favorite feature of Octopus Deploy. Due to my operations background, I see how it can automate all those mundane, time-consuming operations tasks.

Let’s be honest, how many times can you install [IIS](https://www.iis.net/) or [SQL](https://en.wikipedia.org/wiki/Microsoft_SQL_Server) before it becomes tedious, repetitive, and error-prone?

Another benefit of using Chocolatey in Octopus Runbooks is that it helps enable self-service when it comes to applications. You can grant people access to the Octopus project and the runbook so they can run their own installation of Chocolatey packages.

You can use [runbooks](https://octopus.com/docs/runbooks) to automate this part so you can focus on more interesting problems:

- [Routine Operations](https://octopus.com/docs/runbooks/runbook-examples/routine)
- [Emergency Operations](https://octopus.com/docs/runbooks/runbook-examples/emergency)
- [Database Operations](https://octopus.com/docs/runbooks/runbook-examples/databases)
- [AWS Operations](https://octopus.com/docs/runbooks/runbook-examples/aws)
- [Azure Operations](https://octopus.com/docs/runbooks/runbook-examples/azure)
- [GCP Operations](https://octopus.com/docs/runbooks/runbook-examples/gcp)
- [Terraform](https://octopus.com/docs/runbooks/runbook-examples/terraform)

## Tools to install the OS

This blog will not deal with installing the Operating System on your windows server. There are lots of great tools out there that you can use to load the latest Windows Desktop and Server Operating Systems, and I am going to assume you’ve added the [Octopus Tentacle](https://octopus.com/docs/infrastructure/deployment-targets/windows-targets) as part of this process. Some tools I’ve used in the past when prepping windows servers are:

- [Microsoft System Center Configuration Manager](https://en.wikipedia.org/wiki/Microsoft_System_Center_Configuration_Manager)
- [Windows Deployment Services](https://docs.microsoft.com/en-us/windows/deployment/windows-deployment-scenarios-and-tools)
- [Packer](https://www.packer.io/)

Generally, public cloud providers also provide tools to allow you to do Infrastructure as Code such as:

- [Azure Resource Manager Templates](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/overview)
- [AWS Cloudformation](https://aws.amazon.com/cloudformation/)
- [Google Deployment Manager](https://cloud.google.com/deployment-manager/docs)
- [Terraform](https://www.terraform.io/)

### Preparation

Before you start, you need to install an [Octopus Tentacle](https://octopus.com/docs/infrastructure/deployment-targets/windows-targets) on your server.

I'm using the Octopus [Samples instance](https://samples.octopus.app), the next thing I did was add a new space called **Target - Windows**.

As part of the new space configuration, I created:

- An Octopus [environment](https://octopus.com/docs/infrastructure/environments) named **Provisioning**:

![Adding an Octopus environment](images/environment.png "width=500")

- Added a windows server as a [deployment target](https://octopus.com/docs/infrastructure/deployment-targets) and assigned it the [target role](https://octopus.com/docs/infrastructure/deployment-targets#target-roles) **Computer**. I used a [Polling Tentacle](https://octopus.com/docs/infrastructure/deployment-targets/windows-targets/tentacle-communication#polling-tentacles).:

![Adding a deployment target](images/deploymenttarget.png "width=500")

- Checked the Infrastructure tab showed my windows server and that it’s healthy:

![A Healthy deployment target](images/healthy.png "width=500")

- Created a [project](https://octopus.com/docs/projects) called **Computer Provisioning**:

![Adding an Octopus project](images/project.png "width=500")

- Created a [Lifecycle](https://octopus.com/docs/getting-started-guides/lifecycle) called **Computer Lifecycle** and added just the **Provisioning** environment to it, and then assigned it to the project:

![Adding a Provisioning lifecycle](images/lifecycle.png "width=500")

## Runbook configuration

The first thing you need to do is browse to the project and add the runbook:

![Adding a runbook](images/addrunbook.png "width=500")

I created a runbook called **Install Developer Machine Dependencies**:

![Naming the runbook](images/namedrunbook.png "width=500")

### Set timezone, input, and region

One thing that annoys me about setting up Windows is having to configure my non-default regions, which isn’t the US. I use a PowerShell script to set this for all servers. The below works for me, and you can tweak it to your requirements:

```PowerShell
#Set home location to the United Kingdom
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

A US equivalent, on the East Coast, would look something like:

```PowerShell
#Set home location to the United States
Set-WinHomeLocation 0xf4

#override language list with just English US
$1 = New-WinUserLanguageList en-US
$1[0].Handwriting = 1
Set-WinUserLanguageList $1 -force

#Set system local
Set-WinSystemLocale en-US

#Set the timezone
Set-TimeZone "Eastern Time Zone"
```

### Check if Chocolatey is installed

Next, I used was a community-contributed step template called [Chocolatey - Ensure Installed](https://library.octopus.com/step-templates/c364b0a5-a0b7-48f8-a1a4-35e9f54a82d3/actiontemplate-chocolatey-ensure-installed). This step’s only purpose is to check if Chocolatey is installed and to install if it’s not.

![Installing Chocolatey](images/chocolateyinstallstep.png "width=500")

### Install Chocolatey package step

Paul Broadwith of Chocolatey recently updated the [Chocolatey community step template](https://library.octopus.com/step-templates/b2385b12-e5b5-440f-bed8-6598c29b2528/actiontemplate-chocolatey-install-package) to install all of the Chocolatey packages in a single step.

The applications I needed on the windows server are:

```PowerShell
git vscode sql-server-management-studio slack github-desktop rdmfree googlechrome firefox dotnetfx dotnetcore 7zip visualstudio2019professional nordvpn lastpass-chrome lastpass docker-desktop chromium googledrive google-drive-file-stream kubernetes-helm kubernetes-cli minikube zoom notepadplusplus nugetpackageexplorer sdio virtualbox jre8 vlc python foxitreader putty.install sysinternals snagit vagrant packer terraform
```

![Chocolatey Package install step](images/chocoinstall.png "width=500")

### Install Chocolatey Package step parameters

The following parameters are available:

- **Version (Optional)**: You can use this to specify a specific version of the software you want to install. If you’re using more than one package per step and want to set particular software versions, you will need to configure that Chocolatey install and add the version in an additional step.
- **Cache Location (Optional)**: You can use this to specify a cache’s non-default location. I found this useful when installing SQL without having the Tentacle run as an administrator. I found SQL can be a little tricky to install without running the Tentacle Service as a local administrator. You can specify a folder such as `C:\Octopus\Applications` as the cache, which the Local System User has full access to.
- **Package Source (Optional)**: This is probably the *single most crucial parameter in this step*. If you’re doing this at home, it might be acceptable to use the Chocolatey Package Repository, which is the default setting. However, if you’re doing this for a company, please consider using your own package source repositories, such as [Nexus](https://www.sonatype.com/nexus/repository-pro), [Artifactory](https://jfrog.com/artifactory/), or [MyGet](https://www.myget.org/).

The Chocolatey Package resource is built by the community for the community. If you’re using the community repository for Enterprise or large scale package installation, you will likely be [rate limited](https://chocolatey.org/docs/community-packages-disclaimer#rate-limiting). Be careful, and be kind to the community.

The last two options cover whether you want to see the download progress in your logs. Generally, I turn them off as this can result in hundreds of thousands of log files. The other is to allow for additional parameters:

![Specifying Chocolatey Parameters](images/chocoparams.png "width=500")

### Installing IIS & dependencies

The next step was to configure IIS and its dependencies. I used our [IIS Runbooks](https://octopus.com/docs/runbooks/runbook-examples/routine/iis-runbooks) examples and ran this install IIS, and all of its features.

### Optional steps

I’m not a fan of the default website in IIS, so I like to remove it by default. I used the community step template called [IIS Website - Delete](https://library.octopus.com/step-templates/a032159b-0742-4982-95f4-59877a31fba3/actiontemplate-iis-website-delete) and then specified Default Web Site. It will now remove the Default Web Site as part of this provisioning runbook in Octopus.

I use [HyperV](https://docs.microsoft.com/en-us/windows-server/virtualization/hyper-v/hyper-v-technology-overview) as hypervisor when I can, and I wanted to enable it as part of the server provisioning process. I used the **Run a Script** built-in template for this task and used PowerShell to enable HyperV:

```PowerShell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

The last thing I wanted to do was to avoid installing all of the Windows updates that had been released since my server had been prepped. I used a community step template called [Windows - Apply Windows Updates](https://library.octopus.com/step-templates/3472f207-3934-44db-a4ac-1390167cf7ed/actiontemplate-windows-apply-windows-updates), which will automatically install and reboot your machine as long as you set the parameter to **True**.

## Publish the runbook

Runbooks can be in either a draft or published state. You need to publish the runbook before you can execute it.

## Run the runbook

You have the space, the project, the lifecycle, the environment, the server, and the runbook in Octopus. The next step is to run and test the runbook to ensure it does what you want it to do.

To run the Runbook:

1. Open the runbook project.
2. Select {{Operations,Runbooks}}.
3. Select the runbook you created.

![Running the runbook](images/run-runbook.png "width=500")

4. Select **Run**.
5. Select the environment.
6. Hit the **Run** button

![Running runbook](images/run-runbook-run.png "width=500")

At this point, you can grab a coffee because it will take a little while to install all of the applications and dependencies. After your coffee run, the runbook should have completed. Your server should now be fully configured and ready instead of going through the pain of next, next, finish installs and application configuration:

![Completed runbook](images/completedrunbook.png "width=500")

At this point, you should see all of the new applications installed on your server.

## Other scenarios

You can see that this is useful for installing applications not just for ops, DevOps, and developers but other job types. You can create additional runbooks for other job types that need different applications. For instance, a business analyst may want [PowerBI](https://powerbi.microsoft.com/en-us/), and a DBA might wish to install [SQL Toolbelt](https://chocolatey.org/packages/SqlToolbelt). You could even allow people access to runbooks to install and configure pre-approved software.

You can also use this approach for all of your servers, so you can install [SQL](https://chocolatey.org/packages/sql-server-2019) on a database server, or [Tomcat](https://chocolatey.org/packages/Tomcat) on a web server.

:::hint
When installing SQL, you will probably want to create your own Chocolatey Package. SQL is a bit tricker to install as it requires an administrator account to install quickly, and you will also want to configure things such as default users, groups, and locations of the database and log files.

When using Octopus to install SQL Developer Edition or SQL Express, you can do it without the Tentacle running as a local administrator. You will still need to use the optional location for the files. Another gotcha is that if you run the install under a Service account with a named service account, by default, that will use that user as the default SQL administrator. You will need to connect with that account to give yourself access.
:::

## Upgrading Chocolatey packages

As you can see, you can use Octopus to install applications via Chocolatey, but software gets patched, new features and security enhancements are added all  the time. What happens when you want or need the latest version of the software you installed using runbooks and Chocolatey?

You [upgrade](https://chocolatey.org/docs/commandsupgrade) it with Chocolatey and runbooks. This is easy to do, and one of my favorite commands on logging in to a server, or for running using a runbook is:

```PowerShell
choco upgrade all -y
```

This command will run on the server, check against the latest version against the Chocolatey repository you have configured, download the new package, and install it. Think of this as almost a Windows Update but for your Chocolatey package. You can set this up using runbooks and using the Deploy a Script step template and using the upgrade command:

![Upgrading Chocolatey apps runbook](images/upgradechocolateyapps.png "width=500")

After you’ve created the runbook, select **Run**, and it will run the Chocolatey script and upgrade all of your applications:

![Upgraded Chocolatey apps](images/upgradechoco.png "width=500")

### Upgrading Chocolatey Packages on a scheduled trigger

As you now have the upgrade Chocolatey runbook, and you know that it’s working, you can publish the runbook and set a schedule to execute the script, much like a [CRON JOB](https://en.wikipedia.org/wiki/Cron) or a [Windows Task Scheduler](https://en.wikipedia.org/wiki/Windows_Task_Scheduler).

To set this up, select the triggers option under Operations, and select **Add scheduled trigger**.

![Add a Scheduled Trigger](images/addscheduledtrigger.png "width=500")

This will take you to add a **New scheduled trigger** page, and you will need to input:

- **Name**
- **Description**
- **Select the Runbook to run on the scheduled trigger**
- **Select the environment**
- **Select a schedule of daily or alternative schedule**
- **Select the Interval at which it should execute**
- **Select the time the schedule should execute**

![Scheduled Trigger](images/scheduledtrigger.png "width=500")

This will trigger daily at the time you set. I selected 12.30 pm as that’s when most people go to lunch.

You can see all of the configuration in this blog on our [samples instance](https://samples.octopus.app) by logging in as Guest and selecting the **Target - Windows** space.

## Conclusion

Octopus Runbooks and Chocolatey work well together and give you a lot of flexibility to help you automate the installation and configuration of servers both on-premises and in the cloud. They take away the need to install thousands or potentially tens of thousands of applications across your organization’s infrastructure. If you want to see this in action, I presented a webinar with [Paul Broadwith](https://blog.pauby.com) from Chocolatey about [Operations automation with Octopus Runbooks and Chocolatey](https://www.youtube.com/watch?v=E0z4QbwTuBg), which demonstrated how easy this is to do with runbooks on cloud infrastructure.
