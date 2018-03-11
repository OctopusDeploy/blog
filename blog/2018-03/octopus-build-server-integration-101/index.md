---
title: "Integration 101: Octopus and Build Servers"
description: "A brief introduction on how to approach your brand new integration between Octopus and your Build Server"
author: dalmiro.granas@octopus.com
visibility: private
published: 2018-03-14
metaImage: metaimage-cloudsaving.png
bannerImage: blogimage-buildserver.png
tags:
 - Integration
 - TeamCity
 - Team Foundation Server

---

In my 3 years providing support to our users at Octopus, *Integrating Octopus with build servers* is probably the subject I answered the most questions about. With this blog post I'm going to try to give a few tips on how you should approach this task if you are starting it from scratch, regardless of the build server technology you are using. 

This blog post is aimed for users that are just starting their journey integrating Octopus into their continuous integration (CI) pipeline. If you already have this up and running, there might not be too much value here for you :)

## In this post

!toc

## Separation of Concerns

In a way, Octopus and every build server technology out there can seem pretty similar at a first glance. A few things in common they have are:

- Both tools belong in CI pipelines and are aimed to help development teams getting their code out there faster and more reliably.
- Both are basically process runners. You define a Build/Deployment process filled with steps, you execute it and magic happens.

But that's pretty much where the similarities stop. If you want to integrate any 2 tools, you need to have a clear image of which task is each tool going to take care of. 

The below are two (very simplified) lists of what each tool's role should be in an ideal CI pipeline.

### What should the build server do?

- **Compile your binaries**. This means running `MSBuild`,`NPM`,`javac`,`dotnet.exe`, etc, and dropping your compiled app to a folder on the build agent.
- **Run tests**. While its true that you can run these anywhere, because at the end of the day its just a CLI app executing tests from a fixture file, most build servers nowadays come with features that improve the overall testing experience quite a lot. Mostly UI sweetness like showing you how many tests failed right next to your build status, or telling you if the test that failed is a new one or not.
- **Pack and Push your app to a repository**. Once the tests passed, create a package with the output of your build and push it to your repository.
- **Call Octopus and tell it to create a Release/Deployment**

###What should Octopus do?

- **Set configuration values in your application before deployment**. Ideally, the content of your package should be deployable to *any* environment in your pipeline, and the only thing that should be different are environment-specific configuration values such as connection strings, passwords, keys to connect to other services, etc. Octopus has a wide set of feature to deal with any configuration file modification that's needs to be done *before* pushing your application to its destination.

- **Deploy your Application**.

  **Provision infrastructure (optional)**. If you need to create an `Azure WebApp`, an `Elastic Beanstalk` instance, scale up a VM set, create an IIS/Tomcat website or anything that's related to setting up **the place** where you'll be putting your compiled code, adding it as one of the first steps in your Octopus deployment process is a very good idea. //maybe mention a thing our two about our awesome infrastructure as code story.

##So how should I start integrating Octopus into my CI pipeline?

One of the approaches that I've seen has helped many of our customers, was to split this task into 3 stages, while keeping in mind the separation of concerns mentioned previously. We are going call these 3 stages **The Build**, **The Deployment** & **The Glue**. Each stage will have its own **end goal** that we are going to focus on.

Now, before we dwelve into each of the stages, I want to mention that stages 1 and 2 can we worked on in any order, because they won't be touching each other until we reach stage 3. This means that if you are working with a teammate on this integration, he could be focusing in stage 1 while you focus in stage 2 (and you could even bet a beer to see who finishes first).

### Stage 1: The build

**End goal:** By the end of this stage, you should have a package of [any of the supported formats](https://octopus.com/docs/packaging-applications/supported-packages) that contains your build output. The package will only be sitting in a folder on your build agent for now.

We are going to split this stage into 2 steps:

**1.1) Get your project building successfully - ** I've seen this step scare many developers, mostly because it forces them to deal with that black box they've been using for a while called "Build Configuration". It's very common in development teams that only 1-2 devs actually know how their build works, and the rest simply click on "Run" and hope for the best.  If you are in the latter group, this might be a good moment to change that situation and pair up with a teammate to learn how your build process works.

To consider this step done, you should be able to get a successful build following these 2 guidelines:

- The build output must be sent to a fixed folder. Every build tool out there has a parameter that allows you to send the output to a directory of your choice. My recommendation is that you send it to a folder called `Build` or `Output` that sits at the root of your build's `WorkDir`
- The content in that folder should be structured exactly like you expect it to be deployed to its destination. For example if your `Azure Web App` expects a `web.config` and an `index.html` file at the root, then those 2 files should also be at the root of this folder.

**1.2) Get your build output packaged up -** In this step you should be packaging up the contents of the `output folder` mentioned in the previous step into a package of [any of the supported formats](https://octopus.com/docs/packaging-applications/supported-packages). The only key recommendation here is that you version the package with the same version number of the build that's creating it. So if you are building the project `MyWebApp` and you are running the build `1.0.6`, your package should end up being `MyWebApp.1.0.6.zip`

![Package version == build version](packageVersion.png)

You can do this in the same build step where you are building your app (if you are using [Cake](https://cakebuild.net/) for example) or you could have a dedicated build step just for it. Unless I'm using something like Cake, I like to have a dedicated step because its easier to spot if the build failed during the packaging process, the build or any other step I'm running.

By the end of this step you should have a package with the same version as your build, and inside of it should be a copy of your compiled application ready to be deployed to its destination(*).

*(&ast;) you might have to manually modify some configuration values if you want to test this manually. I strongly recommend you to do this once just to make sure your packages are good to go for the next phases.*

###Stage 2: The Deployment

**End goal:** By the end of this stage you should be able to create a release in Octopus and trigger a successful deployment of your application from the command line using [Octo.exe](https://octopus.com/docs/api-and-integration).

We are also going to split this stage into a couple of steps:

**2.1) Upload a test package to the built-in repository - ** This idea of this step is simply to get a package with your compiled application on it, so you can use it in the deployment process you are about to setup. If you've already finished **Stage 1**, you can grab one of the packages that was created during your builds. If you haven't finished that stage yet, simply compile your app locally and package the output into a package using [Octo.exe pack](https://octopus.com/docs/packaging-applications/creating-packages/nuget-packages/using-octo.exe)

Once you have the package, push it to the [Octopus built-in repository](https://octopus.com/docs/packaging-applications/package-repositories/pushing-packages-to-the-built-in-repository#PushingpackagestotheBuilt-Inrepository-UsingtheOctopuswebportal) and make sure you can see it in the web portal under `Library -> Packages`

![Package in repository](packageInRepository.png)

You only need 1 package for the next step which you'll be using over and over until you get the Deployment Process right. If your deployment process will be using more than one package, repeat this process for each package you'll be needing.

**2.2) Design your deployment process and run it - ** Now here's where you finally start doing things in Octopus. We won't get into too much detail here, because [that's what our documentation is for](https://octopus.com/docs/deploying-applications). But the overall idea of this step is that you setup your Deployment Process in Octopus using the packages you uploaded in **2.1** and that you are able to run it successfully through the UI.

If this is your first time setting up your Octopus project, odds are this is the step where you'll be spending most of your time. At this point don't bother too much about the version number of the packages you are using or the release number in Octopus. The sole purpose of this step is that you get comfortable with your Deployment Process and that you fully understand what each step brings to the table.

So sit back and trigger as many deployments as you need :)

**2.3) Create a release and trigger a deployment using Octo.exe**

In the previous step you learned how to create a release and trigger a deployment from the Web Portal. The goal of this step is that you learn to do the same thing, but using `Octo.exe`.

If you don't know about this CLI tool, the elevator pitch is that it talks to the [Octopus API](https://octopus.com/docs/api-and-integration/api) and helps you do some of the most frequently used actions against your Octopus Instance.  You can read about all the functionality it provides in [this document](https://octopus.com/docs/api-and-integration/octo.exe-command-line)

The command you should be paying attention to is [create-release](https://octopus.com/docs/api-and-integration/octo.exe-command-line/creating-releases). A few tips about this command:

- If you use the `--deployTo` parameter, it will not only create the release, but also deploy it to an environment. It basically combines the commands `create-release` and `deploy-release`
- use `--progress` to be able to see the deployment log in the console at it executes. Otherwise the command will only create a task in Octopus, and you'll be forced to go to the Web Portal to see how the deployment went.
- use `--whatIf` to see what would happen if you ran that command, without actually triggering anything in Octopus.

Every single build server integration out there (at least the ones built by the Octopus team) is simply a UI wrapper around this CLI tool. So the knowledge you gain from this step will come in really handy on the next stage.

###Stage 3: The Integration

Now this stage is where we'll put together everything we did in the two previous phases. For this reason its necessary that you finish both of them successfully.

**End goal:** By the end of this stage, you should be able to add a new step to your build process that triggers a deployment in Octopus.

If you are using any of the below build servers, then we recommend you to install the plugin/extension mentioned in the link, which will add a few new steps/runners for you to use in your build process. As mentioned before, these new steps will only be UI wrappers around `Octo.exe`, so most of the fields you'll be filling in them should be very similar to the parameters you used in `2.3`

- [VSTS/TFS](https://octopus.com/docs/api-and-integration/tfs-vsts)
- [TeamCity](https://octopus.com/docs/api-and-integration/teamcity)
- [Bamboo](https://octopus.com/docs/api-and-integration/bamboo)
- [Jenkins](https://octopus.com/docs/api-and-integration/jenkins)
- AppVeyor (coming soon)

*The step you should pay attention to will have the words "Create Release" on its name in pretty much all cases*

If you are not using one of the above build servers, don't worry! The knowledge you gained in `2.3` should be more than enough for you to be able to add a Powershell/Bash script step to your build process that runs the same `Octo.exe` command that you already used.

```
:::hint
**Pro Tip**
Every one of the "Create Release" steps provided by the plugins/extensions will have a checkbox called Show deployment progress or similar. If you check that box, what's going to happen is that your build server will keep the connection open with your Octopus deployment, and it will show you the output of it in your build log. Additionally, if your deployment fails, that step in your build process will also fail. This last bit is extremely handy as you'll know by just looking at your build if the deployment in Octopus failed or not.

If you are using a raw Octo.exe call, then the equivalent of this feature is the --progress parameter.
:::
```

---------------

And that's it! I really hope this guide helps you integrate Octopus into your CI pipeline in a more organized fashion. Please keep in mind that