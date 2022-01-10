---
title: Packaging for .NETCore, on .NETCore, with Octopus
description: Using Octopus tooling to package .NETCore applications on .NETCore
author: shannon.lewis@octopus.com
visibility: public
metaImage: metaimage-package-netcore.png
bannerImage: blogimage-package-netcore.png
bannerImageAlt: Octopus Packaging .NET Core banner
published: 2018-08-06
tags:
 - Product
---

![Octopus Packaging .NET Core banner](blogimage-package-netcore.png)

!include <octopus-cli>

Over the past few months, we’ve had a number of questions and requests for better support around building and packaging .NET Core applications. We’ve had support for that for quite a while, but what has been interesting is the number of requests for supporting building .NET Core applications on .NET Core. What does that mean exactly? It means supporting building .NET Core applications on machines that only have .NET Core, and not the full .NET framework.  Think Linux or Mac OS machines.

If that’s a space you’re working in or looking to move into we’ve got some exciting news. Along with all of the other exciting things included in [2018.7](https://octopus.com/blog/octopus-release-2018.7), we’ve updated `octo.exe` so you can now access it as a .NET command-line extension.

## Introducing dotnet octo

For some time now, `octo.exe` has been cross-platform, with support for running on both .NET Framework and .NET Core. However, it’s been awkward executing `octo.exe` while doing a build on a .NET Core only platform.

Enter the `dotnet octo` global tool. It does everything [`octo.exe`](https://octopus.com/docs/octopus-rest-api/octopus-cli) does, but it can be called using `dotnet octo <command>`. This provides a convenient way to get `octo.exe` onto any machine that has the latest dotnet SDK version available.

To work the magic, summon `octo.exe` onto your build machine using the following:
```bash
dotnet tool install Octopus.DotNet.Cli --tool-path /path/to/install
```

Then to awaken `octo.exe` from its slumber invoke classic incantations such as `dotnet octo pack` and `dotnet octo create-release`.

Just as a note on the `tool-path` argument, you can replace that with the --global flag and it will install the tool globally. Depending on your build machine configuration, installing globally possibly isn’t a good idea though, installing locally to the folder the build is running in provides better isolation. Unfortunately, the isolation comes with a minor wrinkle where the dotnet command-line doesn’t provide the same argument to help find the tools you’ve put into custom tool paths. To get around this, you have to **ensure the path is added to the environment Path variable**.

Also note, the above summons the latest version onto your build machine. There is a [version switch](https://docs.microsoft.com/en-us/dotnet/core/tools/dotnet-tool-install) if you want finer control.

So what would a build script that brings all of this magic together look like? Here’s a simplified example:

```bash
dotnet publish MyAwesomeWebApp -o myMarshallingFolder

octo pack --id=MyAwesomeWebApp --version=1.0.0.0 --outFolder=myArtifactsFolder --basePath=myMarshallingFolder

octo push --package=myArtifactsFolder\MyAwesomeWebApp.1.0.0.0.nupkg --server=https://my.octopus.url --apiKey API-XXXXXXXXXXXXXXXX
```

Like I said, this is simplified. When you’re setting this up from your favorite build tool, you might want to split it into three separate steps.

## Runtime and SDK requirements

In order to use Octo as described here, you must use the .NET Core SDK `2.1.300` or newer.

## Build servers

Our TeamCity extension will already handle switching between `octo.exe` and `dotnet octo`, so you shouldn’t need to change anything in your existing steps for `push`, `create-release` etc. You will have to work out a strategy for the `dotnet tool` command. You could run that as a script at the beginning of your build process, or you could have it pre-run on your build agents. Also coming very soon to the TeamCity extension is a separate `pack` step, for those who want to pack and then use a feed other than Octopus’s built-in feed (e.g. TeamCity’s feed or Artifactory).

The **v3.0** update of the VSTS extension includes the updates to support using `dotnet octo`. The changes include a move away from using PowerShell, which makes it compatible with build agents running operating systems like Linux.

## But what about OctoPack?

There’s one more thing to cover in this post. The elephant in the room, if you will. What about OctoPack?

The short answer is that OctoPack relies on some mechanics of NuGet and MSBuild that have changed in the .NETCore world and trying to port it to work in this new world doesn’t seem like it would provide value over using `octo.exe`.

One of the key parts here, is the application formats we now support. Back when OctoPack came to be, it had two key application types to worry about. Web apps and Windows apps. Both of these can be packaged by simply grabbing the binary outputs and any files marked as content (this is what OctoPack does internally when building a nuspec file).

Fast forward to today, and we have application formats like Cloud Services and Service Fabric. Supporting the ever growing number of these formats isn’t practical in OctoPack, so we recommend using the `package` target that’s built into VS/MSBuild. There’s an [example of how to do this](https://octopus.com/docs/deployment-examples/deploying-asp.net-core-web-applications) in our documentation.

The one caveat to this is that `octo.exe` uses `Octopus.Client` for it’s `push` command, so it is limited to only pushing to the Octopus built-in feed. If you need to push to another package service, you will need to use `NuGet.exe` rather than `octo.exe`. We’re looking at options to address this.

## Conclusion

The .NET Core world is still a fast moving place, so this is a step in what I’m sure will be a longer journey. If you’re building .NET Core applications, please give `dotnet octo` a spin and give us feedback below to help guide that journey.

## Learn more

* Guide: [How to deploy an ASP.NET web app to Azure](https://hubs.ly/H0gBSdJ0)
* [Setting up your own cloud-based CI/CD pipeline Using AppVeyor and Octopus to deploy an ASP.NET Core web app](https://hubs.ly/H0gBSdL0)
* [Build Pipelines and Application Packaging With .NETCore](https://hubs.ly/H0gBQDD0)
* Documentation: [Packaging Applications](https://hubs.ly/H0gBQDH0)
* Documentation: [Versioning](https://hubs.ly/H0gBSdQ0)
* [Deploying an ASP.NET Core web app to Linux](https://hubs.ly/H0gBSdV0)
