---
title: Octopus and NuGet in a .NETCore world
description: How Octopus is evolving in the
author: shannon.lewis@octopus.com
visibility: private
published: 2018-06-29
tags:
 - OctoPack
 - NETCore
 - NuGet
---



I while back we posted about [packaging formats](https://octopus.com/blog/wanted-universal-packaging-format) and why Octopus uses the NuGet repository protocol.

In its infancy Octopus started out supporting only NuGet packages. It also provided [OctoPack](https://g.octopushq.com/ExternalToolOctoPack) to help package the type of applications that were being built at the time.

A few years have gone by since that post and the world has changed a bit. In this post we're going to look at how the NuGet ecosystem has evolved, some new features Octopus now supports, and how this all impacts packaging your applications.

## Package granularity is evolving with .NETCore

The NuGet package format has always supported a far richer set of features than Octopus needed to utilize for our purposes. For Octopus, we're looking for a package that contains an application to deploy.

NuGet though was built with package management for sharing libraries in mind. That's what it does, and it does it well.

The appearance of .NET Core has seen somewhat of a evolution in the way the packages tend to be constructed, or more in what they tend to contain. Traditionally it wasn't uncommon to have a number of related Dlls contained in the package. .NET Core has changed this slightly to focus on a single assembly, or rather the platform specific versions of a single assembly.

As a way of illustrating this, imagine you have a Visual Studio solution with an application and some library projects it depends on. If you run 'dotnet pack' on that solution, you don't get a single NuGet package with all of the library Dlls in it, you get a NuGet package per library project with dependencies specified between them.

What you'll also notice is that the application project doesn't get packaged. Why not? Well because it's not a library, why would you want to push it to NuGet?

## Packaging applications

So this brings us to application packaging. Octopus uses NuGet for the reasons cited earlier, but the Visual Studio and other tools doesn't have that focus in mind so doesn't support it.

What that tooling wants you to do is Publish your application. This produces a format that's conducive to transporting the application to its intended target. For example, an ASP.NET web application destined to be WebDeployed to an Azure Web App.

Some of the other application formats are Azure Cloud Services, Azure Functions, and Service Fabric applications.

## Octopus package formats

Over time we have expanded the supported package formats to include formats like Zip and Maven. So our world has expanded beyond the boundaries of NuGet.

So this is all very interesting but where's it leading?

## OctoPack vs `Octo.exe pack`

Those who've used OctoPack will be familiar with what it does, but for this discussions let's dig a little into how it actually does it.

When you add the OctoPack NuGet package reference to your project, the package contains an install.ps1 file that adds some custom targets to the csproj. Those targets hook in after the build target and when you provide `/p:RunOctoPack=true` to MSBuild they kick in.

When they kick in they use some APIs provided by MSBuild (TODO check this) to list the binary output and the content files in the project. This list is used as part of building a nuspec file that is then used to pass to `NuGet.exe` to create a package.

This all works well for a Web application, but for the other formats listed in the previous section it doesn't work. They use their own transforms and all sorts of things during the Publish to produce the correct content in the correct folder structure.

This leads us to 2 problems. The first one is probably more obvious, to make OctoPack support all of these we'd have to essentially recreate what the Publish target is doing so we came up with the correct content.

The second relates to changes in later versions of the NuGet package format, where the install.ps1 isnt' supported and the setup is far more problemmatic (TODO: re-verify this in case it's no longer the case).

Based on all of this, what we've been recommending to customers is to let the build tooling do the build and Publish. That's what it does, and it does it well.

Once it's done its job, let `Octo.exe` do it's job. What's its job? To take a published folder and assemble it into a package. How does it do that? It simply "zips" the folder you point it at. This could be a straight up Zip file, or it could be a NuGet package with the associated metadata. One of `octo.exe`'s advantages is that it supports multiple formats.

## `dotnet octo`

That's awesome, but what about .NET Core and cross platform?

Well, for some time now `Octo.exe` has been cross platform, with support for running on both .NETFramework and .NET Core. What's been awkard until now has been executing octo while doing a build on a .NET Core only platform (think Linux build agent).

Enter the `dotnet octo` CLI extension. It does everything `Octo.exe` does, but can be called using `dotnet octo <command>` which works at either a project level or globally.

In order for `octo` to be made available at the project level you must add it as a tool reference to your project file. Unfortunately this has to be done manually but is quite simple to do as shown in the following minimal project file example:

```
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>netcoreapp2.1</TargetFramework>
  </PropertyGroup>

   <ItemGroup>
    <DotNetCliToolReference Include="Octopus.DotNet.Cli" Version="4.38.0" />
   </ItemGroup>
</Project>
```

This makes octo available via the dotnet command when your current working directory is the project folder. It has the added benefit of being available as soon as the project is restored which is great if you have some build scripts setup to make use of it.

The dotnet SDK `2.1.300` also introduced some new commands which make it easy to [install tools](https://docs.microsoft.com/en-us/dotnet/core/tools/global-tools) such as Octo globally using the `dotnet tool` command i.e.:
```
dotnet tool install -g Octopus.DotNet.Cli
```

Although convenient when running some ad-hoc commands, it's less ideal to install things globally in some environments. Luckily, you can install Octo at the solution level using the `--tool-path` parameter like so:
```
dotnet tool install Octopus.DotNet.Cli --tool-path /path/to/solution
```
This will create an executable in the specified folder called `dotnet-octo` which can be invoked directly without the `dotnet` command. This still performs a `dotnet octo` under the covers, however dotnet will not automatically
discover executables in the current working directory. In order to use the same `dotnet octo` command shown earlier you will need to add the tool location to the current environment path variable. If you are using build scripts however, this should not be needed as the generated executable will suffice.

TODO: Build script examples

## Runtime requirements
In order to use Octo as described here, you must have the .NET Core runtime `2.1.0` or newer installed.

## Conclusion
OctoPack is dead, long live `Octo.exe`
TODO: this section may need some work ;)
