---
title: Automating developer machine setup with Chocolatey
description: Learn how to automate a developer machine setup using Chocolatey
author: bob.walker@octopus.com
visibility: public
metaImage: automate_machine_chocolately.png
bannerImage: automate_machine_chocolately.png
bannerImageAlt: Illustration showing Chocolately packages being installed on a developer machine
published: 2019-08-26
tags:
 - Engineering
---

![Illustration showing Chocolately packages being installed on a developer machine](automate_machine_chocolately.png)

There is nothing quite like a fresh install of Windows, especially on a developer machine. With the hardware we get these days, the computer is screaming fast. However, that feeling of bliss is quickly broken, as it is time to install the applications and frameworks needed. The first step is to crack open IE Edge to download Chrome, Firefox, or any other browser. Next up, find all the websites, then manually download and install all the necessary tooling. All of this reminds me a lot of my university days. Back when I was at university, I would reinstall Windows every six months or so. I went to university back in the days of Windows 9x/ME. It seemed everyone, from websites to TechTV, recommended doing that, and that was around twenty years ago. 

Thankfully, this is 2019, and the tooling available is much more robust than when I was in university. We no longer have to waste hours or an entire day getting everything downloaded and installed. In this post, I'll walk you through how to automate setting up a developer machine using [Chocolatey](https://chocolatey.org/). 

!toc

## Why Chocolatey?

I worked for a variety of companies before joining Octopus Deploy. Every one of them ran into the same question: How do we get developers going quickly? The proposed solutions always fell into one of two buckets: 

- A developer image.
- A heavy-handed tool like [SCCM](https://docs.microsoft.com/en-us/sccm/apps/deploy-use/deploy-applications). 

The first bucket contained the developer image. People talked about developer images like they were better than cold drinks on a hot summer day. Hard drive crashed? No worries, swap out the hard drive, use the image, and the developer is up and running in under an hour. Is a new developer starting? Order a new laptop, install the image, and the developer won't have to set up anything. They'll be ready to go before they walk in the door.

Okay, cool. Except, developer tools release new versions all the time. It feels like a week hasn't gone by without Visual Studio bugging me to update to the latest version. Who will maintain that image? How often is that image going to be updated? What often ended up happening was the developers spent more time updating and uninstalling old tools that it actually slowed them down. It made a lot more sense for a core Windows image, with the latest patches, to be created and let the developers go from there. 

The second bucket contains heavy-handed tools like SCCM. The tools I've used have a web interface where I selected the software I wanted to install on my machine, which sounds great. Until you see your favorite tool missing or the version on the list is three years old. Adding it to the list involved multiple steps, and, in some cases, approvals. Do you want to install Visual Studio Code? Cool, that will take several hours to make its way through the bureaucracy, and the list is never updated because developers take the path of least resistance. Why fight through bureaucracy when it takes less than 10 minutes to download and install the latest version? Tools like SCCM work great for non-developer machines when everyone has to have a standard package of software such as a specific version of Microsoft Office.

Chocolatey falls outside of the two buckets described above. If you are a .NET developer, then you should be familiar with NuGet. Chocolatey is NuGet for Windows. If Linux is your preferred OS, then Chocolatey is the package manager, such as apt or RPM. It falls outside of those two buckets because it is light-weight, a quick script installs it, and it installs the latest version of packages by default. It will also install any dependencies the package needs, such as a hotfix or another package. A package in Chocolatey wraps an MSI. It could be an Octopus Tentacle, Visual Studio, or the .NET Core SDK. 

## Getting started

First, you need to install Chocolatey. You can do that by running the scripts [found here](https://chocolatey.org/install). After you do that, you can start using Chocolatey to install applications. You can find which applications are available by going to the [Chocolatey Package page](https://chocolatey.org/packages). I don't have VLC installed on my computer, so I will install that by typing in: 

```
choco install vlc
``` 

Which causes a prompt to appear:

![](choco-install-vlc-start.png)

That prompt is kind of annoying. However, it does tell me I can avoid it by including the `-y` switch in my command. Let's cancel this and then retry with the `-y` switch.

![](choco-install-vlc-with-switch.png)

Now I have VLC installed on my machine. If I want to update it, I need to run the command:

```
choco upgrade vlc -y
``` 

As you can see, because I just installed it, I have the latest version.

![](choco-upgrade-vlc.png)

## Automating development machine setup

When I first started using Chocolatey, my PowerShell knowledge was lacking, so my scripts looked something like this:

```PowerShell
Write-Host "Installing Google Chrome"
choco install googlechrome -y

Write-Host "Installing Firefox"
choco install firefox -y

Write-Host "Installing Redis"
choco install redis-64 -y

Write-Host "Installing 7-zip"
choco install 7zip -y
```

For a few packages, copy/pasting commands isn't too bad. For 20+ packages, copy/pasting the same command gets old pretty fast. What I'd like to do is put all those applications into a comma-separated list at the top and tear through that list:

```PowerShell
$chocolateyAppList = "googlechrome,firefox,redis-64,7zip,dotnetcore-sdk,dotnetcore-windowshosting"
if ([string]::IsNullOrWhiteSpace($chocolateyAppList) -eq $false){   
    Write-Host "Chocolatey Apps Specified"  
    
    $appsToInstall = $chocolateyAppList -split "," | foreach { "$($_.Trim())" }

    foreach ($app in $appsToInstall)
    {
        Write-Host "Installing $app"
        & choco install $app /y
    }
}
```

### Windows features

If you are a web developer for Windows, there is an excellent chance you will need IIS installed on your machine. IIS is not an MSI, but a Windows feature. The good news is that Chocolatey can install those as well. It does this by leveraging what is known as [DISM, or Deployment Imaging Service Management](https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/what-is-dism). 

To find out what features are available to you, you can run the command:

```
Dism /online /Get-Features
```

![](get-dism-features.png)

The command to install DISM features through Chocolatey is:

```
choco install [Feature Name] /y /source windowsfeatures
```

For example: 

```
choco install IIS-ManagementService -y -source windowsfeatures
```

Using the same logic as above, we can include DISM features in our script:

```PowerShell
$dismAppList = "IIS-ASPNET45,IIS-CertProvider,IIS-ManagementService"

if ([string]::IsNullOrWhiteSpace($dismAppList) -eq $false){
    Write-Host "DISM Features Specified"    

    $appsToInstall = $dismAppList -split "," | foreach { "$($_.Trim())" }

    foreach ($app in $appsToInstall)
    {
        Write-Host "Installing $app"
        & choco install $app /y /source windowsfeatures | Write-Output
    }
}
```

### Visual Studio and other missing applications

At the time of writing this, the latest version of Visual Studio in Chocolatey is Visual Studio 2017. Visual Studio 2019 came out in April of 2019. The lack of the latest Visual Studio highlights the one weakness of the Chocolatey public repository. You are at the mercy of the company who creates the app to create the package or someone from the community to update the package. However, you do have the ability to [create your own packages](https://chocolatey.org/docs/create-packages). You can even set up an [internal repository](https://chocolatey.org/docs/how-to-host-feed) (just like you can with NuGet packages). Internal repositories are free but consider [purchasing Chocolatey](https://chocolatey.org/pricing) for your team or the company. 

**Post Publish Note:** Visual Studio 2019 is up there; my `Google Fu` was weak on the day I wrote the post.  You can find [Visual Studio 2019 Enterprise](https://chocolatey.org/packages/visualstudio2019enterprise), [Visual Studio 2019 Professional](https://chocolatey.org/packages/visualstudio2019professional), and [Visual Studio 2019 Community Edition](https://chocolatey.org/packages/visualstudio2019community).  Because I didn't find Visual Studio 2019, I included a note about that being a weakness of the Chocolatey public repo.  Anyone can publish a package if a company does not, which one could argue makes that a strength, not a weakness.  Which is an excellent counter-point, the ability for anyone to publish a package is a big strength.  There is a reason why there are thousands upon thousands of applications in the public repo.  The takeaway from this is: there is an excellent chance the public repo has the latest and greatest of any tool you will need.  However, there is that small chance the tool of your choice isn't on the public repository.  Especially if the tool is internally built and maintained, it is easy to create a package for that tool.  However, where you publish that package will be based on your company policy and the tool.  Hopefully, you can contribute back to the community and publish the package to the public repo.  But if you can't publish to the public repo, then creating an internal repo is still very much an option.  If you do opt for an internal repo, consider [purchasing Chocolatey](https://chocolatey.org/pricing) for your team or the company.

### Creating a re-usable script

So far, all the script examples had hardcoded variable values for a small team or company that works fine. As more teams use this, you need to provide some flexibility. I've seen several situations where .NET teams in the same company use a different toolset due to the applications they work on. One team might need WIX for a Windows Form application while another team only works on Angular Websites with ASP.NET WebApi back ends. The script should accept parameters.

Another thing to consider is we don't know when the script will be run, and if the person running it already has Chocolatey installed. The script should be able to handle that scenario and install Chocolatey if needed:

```PowerShell
Param(  
    [string]$chocolateyAppList,
    [string]$dismAppList    
)

if ([string]::IsNullOrWhiteSpace($chocolateyAppList) -eq $false -or [string]::IsNullOrWhiteSpace($dismAppList) -eq $false)
{
    try{
        choco config get cacheLocation
    }catch{
        Write-Output "Chocolatey not detected, trying to install now"
        iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
    }
}

if ([string]::IsNullOrWhiteSpace($chocolateyAppList) -eq $false){   
    Write-Host "Chocolatey Apps Specified"  
    
    $appsToInstall = $chocolateyAppList -split "," | foreach { "$($_.Trim())" }

    foreach ($app in $appsToInstall)
    {
        Write-Host "Installing $app"
        & choco install $app /y | Write-Output
    }
}

if ([string]::IsNullOrWhiteSpace($dismAppList) -eq $false){
    Write-Host "DISM Features Specified"    

    $appsToInstall = $dismAppList -split "," | foreach { "$($_.Trim())" }

    foreach ($app in $appsToInstall)
    {
        Write-Host "Installing $app"
        & choco install $app /y /source windowsfeatures | Write-Output
    }
}
```

We can now have a script per team specifying the applications to install:

```PowerShell
$chocolateyAppList = "googlechrome,firefox,redis-64,7zip,dotnetcore-sdk,dotnetcore-windowshosting"
$dismAppList = "IIS-ASPNET45,IIS-CertProvider,IIS-ManagementService"

Invoke-Expression "InstallApps.ps1 ""$chocolateyAppList"" ""$dismAppList"""
```

## Additional usage

Take a look at the above PowerShell scripts and answer this question, is there anything development machine-specific to those scripts? There is not. Now, that is an interesting thought as you can use the script to bootstrap Windows Server machines, not just development machines. Which raises the next question, why do that instead of creating an image or using a tool such as DSC? Portability and ease of configuration. 

Just like developer machines, servers also have different applications which need to be installed. Attempting to create "one image to rule them all" isn't feasible. It's much easier to use a core Windows image with the latest patches and install the necessary applications onto that. Or, if you are using a cloud provider, use one of the Windows images provided. 

You can use Chocolatey and DSC together. Or you can use just one tool. I prefer to use Chocolatey over DSC. The [learning curve for DSC](https://www.red-gate.com/simple-talk/sysadmin/powershell/powershell-desired-state-configuration-the-basics/) is much steeper than Chocolatey. But it is also much more powerful. My advice is to look at the tools available and see which one makes the most sense. 

My team creates and destroys our demo infrastructure each day. At 2 AM CDT, the infrastructure comes online. At 9 PM CDT the infrastructure is destroyed. We are standing up servers, installing the needed components using Chocolatey, and installing Tentacles to get the latest code. The server isn't going to live for a long time, and I don't need the overhead of DSC. 

## Conclusion

As you can see, with Chocolatey it's possible to automate developer machine setup, while at the same time provide flexibility for teams to pick their own tooling. That's why I like using Chocolatey. It is a framework I can shape to my needs, which makes my life easier. In my personal usage, I've seen big-time savings. Prior to doing this, when I would get a new machine, I would spend an hour or so installing Windows updates and then several hours downloading and installing my favorite applications. It was not uncommon to waste a couple of evenings or a whole day doing that. Now that ratio has flipped. I still have to spend an hour or so installing Windows updates, but now I kick off the PowerShell script, and about half an hour later, I am ready to roll. Now my only hassle is keeping my Chocolatey application list up to date.