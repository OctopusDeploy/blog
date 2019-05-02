---
title: "PowerShell and IIS: 20 Practical Examples"
description: Real-world examples tested on Windows 2008 R2 through to 2016. Creating sites, configuring application pools, and examples in both the old WebAdministration module and the newer IISAdministration module.   
author: paul.stovell@octopus.com
visibility: public
tags: 
 - Walkthrough
 - Scripting
 - Ecosystem
published: 2017-03-16
---

At Octopus Deploy, we do a ton of work with IIS. If you add up the deployment telemetry from all of our customers, we've done over a million deployments of web sites and services. We've learned a lot along the way, about both how to use the PowerShell IIS modules and how they work under the hood, as well as how to use them reliably. My goal in this post is to share that knowledge and build a single place that we can point people to when they need to use the PowerShell IIS modules. 

This post covers:

- The `applicationHost.config` file and how IIS configuration works
- The new `IISAdministration` PowerShell module introduced in Windows Server 2016/Windows 10
- The older `WebAdministration` PowerShell module used since Windows 2008
- What we've learned from millions of real-world deployments using the PowerShell IIS modules
- Lots and lots of practical examples, tested on all Windows Server OS's from 2008 to 2016, as well as information about Nano Server

​The source for all of these examples lives in a [GitHub repository](https://github.com/OctopusDeploy/PowerShell-IIS-Examples), and we run the examples automatically against test machines running each of the different Windows Server OS's, so I'm fairly confident that they work. Of course, if you run into any trouble, post an issue to the GitHub repository's issue list - or send us a pull request! :smile: 

There's quite a bit of theory at the start of this post before we get into the practical examples. There are plenty of sites that show how to do basic IIS tasks; my goal by the end of this post is to **make you an expert at automated IIS configuration**, and the examples just put that into context. 

!toc

## How IIS Configuration is stored

If you open up the **IIS Manager** user interface and browse around any web site or application pool, you'll find no shortage of knobs and dials that you can tweak. There are thousands of potential settings, from what authentication methods to use, to how log files should be written, to how often the application pool that runs your process should recycle. 

![IIS Manager, the main user interface for graphical management of IIS](iis-powershell-iis-manager.png "width=500")

Where do all of these settings live? 

Since IIS 7 (Windows 2008), nearly all of these settings live in one of two places:

- The `web.config` file that is deployed with your application
- An `applicationHost.config` file, which is server-wide

If you are an ASP.NET developer you're no-doubt familiar with the `web.config` file, but you might not have seen `applicationHost.config` before. You'll usually find it under:

```
C:\Windows\System32\inetsrv\config\applicationHost.config
```

If you work with IIS, it's worth taking the time to really look around this file, as you'll discover all kinds of interesting settings. For example, here's how your application pools are defined:

```xml
<configuration>
    <!-- snip -->
    <system.applicationHost>
        <!-- snip -->
        <applicationPools>
            <add name="DefaultAppPool" managedRuntimeVersion="v4.0" />
            <add name=".NET v2.0 Classic" managedRuntimeVersion="v2.0" managedPipelineMode="Classic" />
            <add name=".NET v2.0" managedRuntimeVersion="v2.0" />
            <add name=".NET v4.5 Classic" managedRuntimeVersion="v4.0" managedPipelineMode="Classic" />
            <add name=".NET v4.5" managedRuntimeVersion="v4.0" />
            <add name="OctoFX AppPool" autoStart="false" startMode="AlwaysRunning">
                <processModel identityType="LocalSystem" />
            </add>
```

As are the web sites:

```xml
<configuration>
    <!-- snip -->
    <system.applicationHost>
        <!-- snip -->
        <sites>
            <site name="Default Web Site" id="1">
                <application path="/" applicationPool="Default Web Site">
                    <virtualDirectory path="/" physicalPath="C:\inetpub\wwwroot" />
                </application>
                <bindings>
                    <binding protocol="http" bindingInformation="*:80:" />
                </bindings>
            </site>
```

Why is it important to be familiar with `applicationHost.config` file? Well, it comes down to this:

> All of the PowerShell modules for IIS that we discuss below are just fancy wrappers for editing this big XML file. 

If you understand this, you'll find it much easier to understand why the PowerShell IIS modules work the way they do, and you'll be able to work out how to configure settings that aren't obvious. 

## IIS PowerShell Modules and OS Versions

IIS relies heavily on services provided by the Windows kernel, so each version of IIS has been coupled to a release of Windows. And since each version of IIS brings new features, there have been different attempts at providing the PowerShell modules. The table below outlines each of these combinations:

| Operating System           | IIS version | PowerShell modules                       |
| -------------------------- | :---------: | ---------------------------------------- |
| Windows Server 2008        |      7      | `Add-PsSnapIn WebAdministration`         |
| Windows Server 2008 R2     |     7.5     | `Import-Module WebAdministration`        |
| Windows Server 2012        |      8      | `Import-Module WebAdministration`        |
| Windows Server 2012 R2     |     8.5     | `Import-Module WebAdministration`        |
| Windows Server 2016        |     10      | `Import-Module WebAdministration` or <br> `Import-Module IISAdministration` |
| Windows Server 2016 - Nano |     10      | `Import-Module IISAdministration`        |

Let's quickly run through the history. 

### IIS 7 and the WebAdministration "SnapIn"

IIS 6 supported running ASP.NET applications, but was written in unmanaged (non-.NET) code and all modules and extensions were unmanaged. IIS 7 was the first version of IIS to support .NET modules in the pipeline, and the new Integrated pipeline mode. IIS 7 shipped with Windows Server 2008, but <how is PowerShell installed>?

PowerShell 1.0 used SnapIns ("modules" were introduced in PowerShell 2.0), but most of the supported commands are the same as used in the later module versions. So on Windows 2008 (prior to R2) you'd load the PowerShell IIS support with:

```powershell
Add-PsSnapIn WebAdministration
# Use commands like: Get-Website, New-Website, New-WebAppPool
```

What complicated matters is that PowerShell 2.0 is available on Windows Server 2008 as an upgrade, but the PowerShell IIS module isn't - you still have to use the snap-in. 

### IIS 7.5+ and the WebAdministration Module

Windows Server 2008 R2 shipped in 2011 and included PowerShell 2.0 **FC**, which meant the IIS commands were now available as a module. You load them using the following command:

```powershell
Import-Module WebAdministration
# Use commands like: Get-Website, New-Website, New-WebAppPool
```

### IIS 10 and the IISAdministration Module

IIS 10, which ships with Windows Server 2016/Windows 10, provides you with two choices. You can continue to use the old module:

```powershell
Import-Module WebAdministration
# Use commands like: Get-Website, New-Website, New-WebAppPool - same as before
```

Or you can use the [brand new](https://www.iis.net/learn/get-started/whats-new-in-iis-10/iisadministration-powershell-cmdlets) `IISAdministration` module:

```powershell
Import-Module IISAdministration 
# Use commands like: Get-IISSite, New-IISSite, New-IISAppPool
```

Why the change? The diagram below explains how each of the modules are built. 

![Diagram of the PowerShell IIS module architecture choices in Windows Server 2016](iis-powershell-architecture.png "width=500")



Back when IIS 7 shipped, to make it easier to work with `applicationHost.config` from managed code, Microsoft created a .NET library called `Microsoft.Web.Administration.dll`. You'll find it in the GAC, and you can reference it and [use it from C# code](https://www.iis.net/learn/manage/scripting/how-to-use-microsoftwebadministration): 

```csharp
using (var manager = new ServerManager())
{
    foreach (var site in manager.Sites)
    {
        Console.WriteLine("Site: " + site.Name);
    }
}
```

If you look through the class library documentation, you'll see that it's very much a wrapper around the XML configuration file - for example, the `Application` class inherits from a `ConfigurationElement` base class. 

For some reason, the `WebAdministration` PowerShell module never built directly on top of this class library, but instead duplicated much of it. This duplication is pretty obvious when you look at it in a decompiler. 

With the new `IISAdministration` module, Microsoft have basically:

- Rebuilt it on top of the `Microsoft.Web.Administration` DLL
- Removed the `IIS:\` drive PowerShell provider
- Created new `IIS*` cmdlets that provide a "jumping point" into the `Microsoft.Web.Administration` classes

As users, the `IISAdministration` version of the module has one big advantage - the types returned by the commands are much more useful. While they are still XML wrappers, they are adorned with useful properties and methods that make the objects much more usable without relying on the commands:

```powershell
Import-Module IISAdministration
$site = Get-IISSite -Name "Default Web Site"
$site.ServerAutoStart = $true   # Make the website start when the server starts
$site.Applications.Count        # How many applications belong to the site?
$site.Bindings[0].Protocol      # Get the protocol (HTTP/HTTPS) of the first binding
```

If you're using the old `WebAdministration`, you pretty much have to do everything with the PowerShell CmdLets or by embracing the XML. There's no way to navigate from a site returned by `Get-Website` to its applications, as one simple example. The new module also works much better with PowerShell pipelining. 

Overall, while I think the refactoring Microsoft did in the new module is a good move, I wish it wasn't such a breaking change. I can imagine this simplifies maintenance for them - instead of building a first-class PowerShell/IIS experience, they've built a handful of cmdlets that return .NET objects, and then you make standard method calls on them. 

### Nano Server broke everything!

Maybe that's a bit harsh, but there's some truth to it. It goes like this:

1. Nano Server is a very cut-down version of Windows designed for containers or as a VM - it's a few hundred MB and boots instantly. [I'm pretty excited about it](https://octopus.com/blog/nano-server-future-deployment-models).
2. Nano Server can't run the full .NET Framework - it can only run .NET Core
3. PowerShell was built on the .NET Framework. 
4. To be able to run PowerShell on Nano Server (and Linux!), they rebuilt PowerShell on .NET Core
5. The old `WebAdministration` module also required the full .NET Framework. They ported a version of `Microsoft.Web.Administration.dll` to .NET Core (after all, it's just a big XML wrapper) so the `IISAdministration` module could work, but they never ported the `WebAdministration` module. 
6. Microsoft have [no plans to port WebAdministration to Nano Server](https://www.iis.net/learn/get-started/whats-new-in-iis-10/introducing-iis-on-nano-server)

This means that if your script needs to run on Nano Server, you're limited to only using the `IISAdministration` module - the old `WebAdministration` module that we've come to know and love no longer works. 

![Nano Server supports IISAdministration, but not WebAdministration](iis-powershell-nano.png)

### Writing scripts that run on multiple OS's

At Octopus, our code to deploy IIS websites has to support all of these operating systems. The simplest and most reliable way that we've found to load the `WebAdministration` module is no matter what OS you're on is:

```powershell
Add-PSSnapin WebAdministration -ErrorAction SilentlyContinue
Import-Module WebAdministration -ErrorAction SilentlyContinue
# Use commands like: Get-Website, New-Website, New-WebAppPool
```

If you want to be sure you've loaded at least one of the modules (IIS may not be installed), you can [see how we do it in Octopus](https://github.com/OctopusDeploy/Calamari/blob/master/source/Calamari/Scripts/Octopus.Features.IISWebSite_BeforePostDeploy.ps1#L33-L45). 

A script written this way should be portable between OS's, unless you are running it on Nano Server. In that case, you'll need to use the new module, and the `IIS*` CmdLets instead. 

## Recap IIS Theory

### Sites, Applications and Virtual Directories

We're going to use these terms a lot throughout this post, so I think it's worth a slight recap. Let's take this IIS server:

![IIS server with many sites, applications and virtual directories](iis-powershell-sites-apps-vdirs.png)



Here we have one IIS server, which serves multiple **web sites** (Website1 and Website2). Each website has bindings that specify what protocol (HTTP/HTTPS, sometimes others), port (80/443/other) and host headers it listens on. 

Website2 also has many **applications** (App1, App1.1, App1.2, App2), and **virtual directories** (App.1.1.vdir1, Vdir1, Vdir1.1, Vdir2). 

Don't let that screenshot from IIS Manager fool you. Under the hood, IIS thinks about the relationship between these sites, applications and virtual directories quite differently. From [the IIS team](https://www.iis.net/learn/get-started/planning-your-iis-architecture/understanding-sites-applications-and-virtual-directories-on-iis):

> Briefly, a site contains one or more applications, an application contains one or more virtual directories, and a virtual directory maps to a physical directory on a computer.

This may take a few minutes to wrap your head around, but it's important. IIS Manager presents the tree above because that's how users think of their sites, apps and vdirs, but here's how IIS *actually* models the data:

- Site: **Website1**
   - Application: **/**
     - Virtual directory: **/**
- Site: **Website2**
   - Application: **/**
     - Virtual directory: **/**
     - Virtual directory: **/Vdir1**
     - Virtual directory: **/Vdir1/Vdir1.1**
     - Virtual directory: **/Vdir2**
   - Application: **/App1**
     - Virtual directory: **/**
   - Application: **/App1/App1.1**
     - Virtual directory: **/**
     - Virtual directory: **/App1.1.vdir1**
   - Application: **/App1/App1.1/App1.2**
     - Virtual directory: **/**
   - Application: **/App2**
     - Virtual directory: **/**
   - Application: **/Vdir1/Vdir1.App3**
     - Virtual directory: **/**

This is the object model that the Microsoft.Web.Administration .NET assembly presents, but it's also the model used by the XML in `applicationHost.config`:

```xml
<sites>
    <site name="Website2" id="2">
        <application path="/" applicationPool="MyAppPool">
            <virtualDirectory path="/" physicalPath="..." />
            <virtualDirectory path="/Vdir1" physicalPath="..." />
            <virtualDirectory path="/Vdir1/Vdir1.1" physicalPath="..." />
            <virtualDirectory path="/Vdir2" physicalPath="..." />
        </application>
        <application path="/App1" applicationPool="MyAppPool">
            <virtualDirectory path="/" physicalPath="..." />
        </application>
        <application path="/App1/App1.1" applicationPool="MyAppPool">
            <virtualDirectory path="/" physicalPath="..." />
            <virtualDirectory path="/App1.1.vdir1" physicalPath="..." />
        </application>
        <application path="/App1/App1.1/App1.2" applicationPool="MyAppPool">
            <virtualDirectory path="/" physicalPath="..." />
        </application>
        <application path="/App2" applicationPool="MyAppPool">
            <virtualDirectory path="/" physicalPath="..." />
        </application>
        <application path="/Vdir1/Vdir1.App3" applicationPool="MyAppPool">
            <virtualDirectory path="/" physicalPath="..." />
        </application>
        ...
```

Some notes:

- In IIS, the object model isn't as crazy as the tree in IIS Manager presents - Sites have Applications. Applications have virtual directories. That's that. Deeply nested relationships are modelled by storing the paths. 
- Even though IIS Manager shows VDir1 as having an application inside it, the reality is that the application belongs to the site. 
- Website1 is just a site, without any applications or virtual directories - but that's just IIS Manager making life easy for us again. There's actually an application and a virtual directory - they just use "/" as the path. 
- The physical path for all these things is actually defined by the virtual directory. 

The **WebAdministration** PowerShell module actually goes to quite some effort to hide this - you navigate IIS similar to IIS Manager. But the new **IISAdministration** module gives up on that leaky abstraction, and presents the world to you this way. 

### Location sections

One of the confusing behaviors of IIS configuration is that as you make changes in IIS Manager, different settings get stored in different places:

- In the `<sites>` element shown above
- In `<location>` sections inside applicationHost.config
- In your own web.config file

For example, when you change the physical path or bindings of a website, that's stored in `<sites>`. 

When you change settings like whether directory browsing is enabled, that goes to the `web.config` file in the physical path of whatever virtual directory (and remember, sites and apps all have virtual directories!). 

When you change what authentication modes (anonymous, basic, Windows, etc.) should apply, that's written to a `<location>` section at the bottom of `applicationHost.config`. 

```xml
<location path="Website2/Vdir1">
    <system.webServer>
        <security>
            <authentication>
                <anonymousAuthentication enabled="false" />
            </authentication>
        </security>
    </system.webServer>
</location> 
```
The rule that seems to apply is:

1. If it's a setting that should be "local" to the thing it applies to, it's stored in `<sites>`. For example, when I change the application pool of **Website2**, I wouldn't expect it to also change the application pool assigned to the applications inside it. 
2. If it's a setting that app developers are likely to want to set themselves, it goes to web.config.
3. If it's a setting that would be set by IIS administrators, and not by app developers, but it's something they would expect to *inherit down the paths*, then it's set with the `<location>` in `applicationHost.config`. For example, if I disable anonymous authentication at the root website, I would expect that to apply to everything underneath it. If I then re-enable it for one virtual directory, I'd expect other apps and virtual directories under that path to inherit the new value.  

It's worth noting that you actually have some control over rule #3. IIS "locks" certain settings - the authentication settings for example - so that they can't be overridden by a naughty application developer. But you can [unlock them yourself and allow individual apps to override them in their own web.config files](https://www.iis.net/learn/get-started/planning-for-security/how-to-use-locking-in-iis-configuration#Task1). 

The easiest thing to do is: make the change in IIS Manager, then go looking for whether it was applicationHost.config or your web.config that changed. 

## IIS:\ drive provider vs. CmdLets

There are two supported paradigms for working with the PowerShell IIS modules:

- The `IIS:\` drive PowerShell provider, which lets you work with IIS as if it were a file system
- The task-based helper cmdlets, like `New-Website`

In fact, if you decompile the cmdlets, most of the cmdlets actually wrap the IIS drive approach. When you call `New-Website`, the cmdlet actually does this:

```powershell
# When you call this
New-Website -Name "MySite" -Port 8080 -PhysicalPath "C:\Test"

# It actually generates and calls:
New-Item -Path "IIS:\Sites\MySite" -Type Site -Bindings @{protocol="http";bindingInformation="*:8080:"}
Set-ItemProperty -Path "IIS:\Sites\MySite" -name PhysicalPath -value "C:\Test"
```

At Octopus, we've found that while we often start with the cmdlet approach, our scripts generally all turn into using the `IIS` drive approach, because it allows for more advanced settings. That's why most of the examples shown below use this approach. 

Note that the `IIS:\` drive approach is **not supported by the IISAdministration** module - it's effectively obsolete. 

## Retry, retry, retry

As I've discussed, all of the PowerShell IIS modules are really just wrappers over the XML file. And files don't do very well when multiple processes read or write to them. All kinds of things can lock the file - virus scanners, backup solutions, IIS restarting, someone using the IIS Manager UI. At Octopus for a while one of our most persistent support issues was problems that came up when the file was locked. 

In practice, the only solution that has worked for us reliably across millions of customer deployments - and gotten rid of the support complaints - is to retry. Lots. Here's how we do it. First, we make a function that can execute a block of PowerShell with a retry:

```powershell
function Execute-WithRetry([ScriptBlock] $command) {
	$attemptCount = 0
	$operationIncomplete = $true
    $maxFailures = 5

	while ($operationIncomplete -and $attemptCount -lt $maxFailures) {
		$attemptCount = ($attemptCount + 1)

		if ($attemptCount -ge 2) {
			Write-Host "Waiting for $sleepBetweenFailures seconds before retrying..."
			Start-Sleep -s $sleepBetweenFailures
			Write-Host "Retrying..."
		}

		try {
		    # Call the script block
			& $command

			$operationIncomplete = $false
		} catch [System.Exception] {
			if ($attemptCount -lt ($maxFailures)) {
				Write-Host ("Attempt $attemptCount of $maxFailures failed: " + $_.Exception.Message)
			} else {
				throw
			}
		}
	}
}
```

Then, each action we perform with IIS is wrapped in this function:

```powershell
# Start App Pool
Execute-WithRetry { 
	$state = Get-WebAppPoolState $applicationPoolName
	if ($state.Value -eq "Stopped") {
		Write-Host "Application pool is stopped. Attempting to start..."
		Start-WebAppPool $applicationPoolName
	}
}
```

## Examples

The rest of this post will be used to show lots of real-world examples of how to use the PowerShell IIS modules. 

!partial <examples>

## Summary

If you're setting out to automate your IIS deployments for the first time, I hope you'll find the background information as well as the examples in this post useful. As I mentioned, you'll find all of these examples in [a GitHub repository](https://github.com/OctopusDeploy/PowerShell-IIS-Examples), and we run them against Windows 2008 R2 up to Windows Server 2016. If you find any problems or have ideas on other examples this post should include, let me know in the comments or send a pull request! 

## Learn more

* Using Octopus [output variables](http://bit.ly/2RRKCVl) in subsequent deployment steps
* [How to invoke an executable from PowerShell with a dynamic number of parameters](http://bit.ly/2N0MDxQ)
* Tips and tricks for dealing with [PowerShell exit codes](http://bit.ly/2UR5DS8)
* Configuration Management with Octopus and [PowerShell DSC](http://bit.ly/2N0NkqW)
* Documentation: [Running an Azure PowerShell script step](http://bit.ly/2GATRXT)