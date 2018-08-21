---
title: Packaging .NETCore applications
description: Using Octopus tooling to package .NETCore applications
author: shannon.lewis@octopus.com
visibility: private
metaImage: metaimage-package-netcore.png
bannerImage: blogimage-package-netcore.png
published: 2018-08-24
tags:
 - OctoExe
 - .NET Core
 - NuGet
---

![Octopus Packaging .NET Core banner](blogimage-package-netcore.png)

We recently posted an introduction to our newly added capability for [packaging .NET Core applications on .NET Core](octopus-and-netcore.md). In this post we're going to get our hands dirty and dig into how to put together a build pipeline for a .NET Core application. The concepts apply equally regardless of whether the build machine itself has full .NET Framework or .NET Core. 

Along the way we're actually going to build the pipeline a few times over, using different build servers and tools.

## Link 1 in the chain

The focus in this post is to illustrate the build setup, so we're going to assume that you are familiar with everything required to get you to the point of building. We're using a Git repo with a simple ASP.NET Web application for illustration, but the content and mechanics of the application aren't important to the overall story.

## The 3 steps to success

So what is important? In all of the following examples there is 1 underlying pattern being used. The 3 core steps are:

1. publish the application to a folder
2. package the application
3. push the package to a feed that can be used by Octopus

For those familiar with [OctoPack](https://g.octopushq.com/ExternalToolOctoPack), it also uses these same 3 steps, it just does them internally so you don't need to worry about it. As mentioned in the previous post though, it makes some assumptions about the way the application can be published and those assumptions don't hold for some newer application types.

The Microsoft build tooling does understand these application types and their eccentricities. It is also designed to publish the minimal set of binaries and content ready for transport to where it's going to actually execute.

## TeamCity

Alright, without further ado, let's get into our first example. Here's a sneak peek at what our build steps are going to look like when we're finished.

![TeamCity build steps](netcorebuilds\tc-steps.png)

That might look like a lot of steps at first glance, but TeamCity can do some heavy lifting here. Once you've created your project and configured your VCS root go to the Build Steps and let TeamCity have a stab at creating the build steps based on what it can see in the repo. In our sample that got us the first 3 steps in a couple of button clicks.

Now let's fine tune the publish step a little by setting a specific Output directory, and this rounds out publishing to a folder. Note that the output directory is relative to the csproj folder location. I like to use a folder off the root level of the repo, thus the `../../`.

![TeamCity publish step](netcorebuilds\tc-publish.png)

This gets us the application contents published, next we package it.

![TeamCity Octo pack step](netcorebuilds\tc-pack.png)

This step is a recent addition to our TeamCity extension, so if you can't see it you'll need to update the extension. The _Package ID_ is declared as a TeamCity variable because we're going to use it again in the Push step shortly, the actual value isn't important for our illustration.

The _Source path_ is the folder we published to in the previous step and the _Output  path_ is where we're going to put the resulting package. 

Just to reiterate, the actual values used for almost all of these fields aren't particularly important, so long as you understand which fields on each step need to align. The _Package version_ is generally always going to be `%build.number%` though :)

Ok, we're on the home stretch, let's push the package to a feed Octopus can use. You have 2 options at this point, depending on whether you want to use TeamCity's feed as an external feed from Octopus or if you want to use Octopus' internal feed. For the former, you simply tick the _Publish packages as build artifacts_ and you are done.

For the latter you add an `OctopusDeploy: PushPackages` step.

![TeamCity Octo push step](netcorebuilds\tc-push.png)

The _Package paths_ here is built up from the folder that was used as the _Output path_ in the pack step, along with the specific filename of the package. You can wildcard the filename (e.g. `artifacts/*.nupkg`) but we don't tend to recommend that. If the folder doesn't get cleaned up reliably between builds you'll end up trying to push packages from previous builds.

So that's it, you've packaged you're .NET Core application and it's in a feed where Octopus can use it. Time to add an `OctopusDeploy: Create release` step and sit back and watch the deployments roll out. Well, maybe... ;)

### Build chains

There can be a catch if you are using the TeamCity feed as an external feed to Octopus. Before we explain the details, let's have a look at the create release step

![TeamCity create release step](netcorebuilds\tc-rel-step.png)

There are 2 things I've done quite specifically in setting up this step to make sure things go bang if the TeamCity and Octopus worlds aren't correctly aligned. The first is to leave the _Release number_ blank in TeamCity and configure the project settings in Octopus to use the package version, in the _Release Versioning_ setting. The second is to explicitly specify the exact package version that we just pushed to the feed as the package version the release must use, which we do through the additional arguments (yes, in hindsight this might have been better as a 1st class field and we might look at that in the future).

Which brings us to the catch, if you trigger the release the package may not be available in the feed yet and Octopus won't be able to find it. By providing the specific version of the package we will force a build failure here if the package isn't available. Without this the release creation will default to auto selecting the most recent package and everything will look hunky dory until you realize some fix you thought had been released isn't actually there. Many hours of head scratching will ensure until you happen to notice the single digit difference in the package version to the release version.

So why did I also leave the _Release number_ blank in TeamCity, when you'd often see it set to `%build.number%`? Well if I forget to set the version explicitly via the additional parameters then the release in Octopus will end up with the version of the older package that got auto selected, this will either hard fail if that release already exists or will stand out when you look at the project overview. I.e. an incorrect release number is big and bold and much easier to spot in the UI than an incorrect package version within the release.

Ok, now those are good counter measures to have in place, but they just warn us if it all goes wrong, how do we make sure it doesn't all go wrong? The best bet we've found is [Build chains](https://confluence.jetbrains.com/display/TCD10/Build+Chain). I'll leave most of how to deal with build chains as an exercise for the reader, but here's the setup I've used for a "Release" Build Configuration in my demo project.

![TeamCity chained build general settings](netcorebuilds\tc-rel-gen.png)

![TeamCity chained build dependencies](netcorebuilds\tc-rel-dep.png)

![TeamCity chained build triggers](netcorebuilds\tc-rel-trig.png)

This build configuration depends on the "Build" build configuration, and is triggered when that succeeds. This build's version number is also directly set to the build number from its dependency.

## VSTS

Ok, round 2 here we come. I've used the same source code for this setup, so it's literally the same app. The only real difference in setup is I decided to use a different Id for the package I was creating, just to make sure I didn't get version conflicts when I was pushing the packages to my Octopus server.

Like before, here's the sneak peak at the pipeline and then we'll dig into details.

![VSTS Pipeline](netcorebuilds\vsts-pipeline.png)

I've mentioned a couple of times now that the .NET tooling knows the right way to do this stuff. In the pipeline above all I did was ask for a new pipeline and select the .NET Core template from the list. It created the first 5 steps, the 5th being the one I've disabled. I disabled this step rather than deleting it to illustrate the pattern again. By default VSTS wants you to publish to a folder and then use that as the basis for what to move to the target of the deployment, just like we've been talking about. All we're doing here is replacing the Publish Artifact step with our Pack and Push steps, which package the folder content and push the package to Octopus just like we saw in the TeamCity example.

Here's what's in the Publish step. I haven't changed anything in this step, I'm showing it here because the output folder it's using is important for _Source Path_ in the next step.

![VSTS Publish](netcorebuilds\vsts-publish.png)

Once we've got the output folder then it's time to set up the packaging step.

![VSTS Pack](netcorebuilds\vsts-pack.png)

The Publish Artifact step that I disabled was using a folder called `drop` for its output, so I followed suit for simplicity.

![VSTS Push](netcorebuilds\vsts-push.png)

This then feeds in to the final step, to push the package to Octopus. The Octopus Url and API key are taken care of using a service connection in VSTS, which you select from the dropdown on the step.

The only other point to note is the `$(Build.BuildNumber)`. This is a built in variable but you can control it's format (using the `Options` tab that you can see up near the top left). I've changed from the default to `1.0.0$(rev:.r)` for doing this demo. The default scheme works, but I tend to use it with caution. It creates SemVer versions with a major based on the current date, which isn't really in the spirit of SemVer. Do you make breaking changes every day? It's ok, you don't need to answer that ;) The other catch is that if you ever decide to move off this scheme the existing major numbers will be really big, so almost any other scheme would result in versions that Octopus would interpret as older than the existing ones.

## Cake script



## Wrapping Up

.

