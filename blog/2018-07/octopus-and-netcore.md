---
title: Packaging for .NETCore, on .NETCore, with Octopus
description: Using Octopus tooling to package .NETCore applications, on .NETCore
author: shannon.lewis@octopus.com
visibility: private
published: 2018-07-19
tags:
 - OctoExe
 - .NET Core
 - NuGet
---


In the past months we've had a number of questions and requests for better support around building and packaging .NET Core applications. We've had support for that for quite a while, what has been interesting though  has been the number of requests for supporting building .NET Core applications on .NET Core. What does that mean exactly? It means supporting building .NET Core applications on machines that only have .NET Core, and not the full .NET framework. Think Linux or Mac OS machines.

If that's a space you're working in or looking to move into, we've got some exciting news. Along with all of the other exciting things included in 2018.7, we've updated `octo.exe` so you can now access it as a .NET Command line extension.

## Introducing `dotnet octo`

For some time now `octo.exe` has been cross platform, with support for running on both .NET Framework and .NET Core. What's been awkward has been executing `octo.exe` while doing a build on a .NET Core only platform.

Enter the `dotnet octo` global tool. It does everything [`octo.exe`](https://octopus.com/docs/api-and-integration/octo.exe-command-line) does, but can be called using `dotnet octo <command>`. This provides a convenient way to get `octo.exe` onto any machine that has the latest dotnet SDK version available. 

To work the magic summon `octo.exe` onto your build machine using the following:
```bash
dotnet tool install -g Octopus.DotNet.Cli --tool-path /path/to/install
```

To then awaken `octo.exe` from it's slumber invoke classic incantations such as `dotnet octo pack` and `dotnet octo create-release`. 

Just as a note on the `tool-path` argument, you can omit that and it will install the tool globally. Depending on your build machine configuration, installing globally possibly isn't a recommended idea though, installing locally to the user running the build or to the folder the build is running in provides better isolation.

Unfortunately you get better isolation but with a minor wrinkle. The dotnet command line doesn't provide the same argument to help find the tools you've put into custom tool paths. To get around this you have to **ensure the path is added to the environment Path variable**.

So what would a build script look like to bring all this magic together? Here's a simplified example.
```bash
dotnet publish MyAwesomeWebApp -o myMarshallingFolder

dotnet octo pack --id=MyAwesomeWebApp --version=1.0.0.0 --outFolder=myArtifactsFolder --basePath=myMarshallingFolder

dotnet octo push --package=MyAwesomeWebApp.1.0.0.0.nupkg --server=https://my.octopus.url --apiKey API-XXXXXXXXXXXXXXXX
```

Like I said, this is simplified. When you're setting this up from your favorite build tool, you might want to split it into 3 separate steps.

## Runtime and SDK requirements

In order to use Octo as described here, you must use the .NET Core SDK `2.1.300` or newer.

## The elephant in the room
There's one more thing to cover in this post. The elephant in the room, if you will. What about OctoPack?

The short answer is that OctoPack relies on some mechanics of NuGet and MSBuild that have changed in the .NETCore world, and trying to port it to work in this new world doesn't seem like it would provide value over using `octo.exe`.

One of the key part of this thinking is the application formats we now support. Back when OctoPack came to be it had 2 key application types. Web apps and Windows apps. Both of these can be packaged by simply grabbing the binary outputs and any files marked as content (this is what OctoPack does internally when building a nuspec file).

Fast forward to today and we have application formats like Cloud Services and Service Fabric. These are somewhat more complicated than simply binaries and content, so we have 2 choices. First is to recommend using the `package` target that's built in to the VS/MSBuild. Second is to reverse engineer everything that's going on inside those package targets so OctoPack can mimick them. Can you tell which way we're leaning on this?

There is one thing that OctoPack does do that `octo.exe` currently doesn't. It uses `NuGet.exe` under the hood for the `push` command. This means it can push to any NuGet compatible feed. `octo.exe` is built on top of the `Octopus.Client` library, and use that to do a push, so can only push to our feed (the built in feed accepts NuGet packages but doesn't implement the whole NuGet repository API). We're looking at options to address this.

## Conclusion
OctoPack is dead, long live `octo.exe`
TODO: this section may need some work ;)
