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

<div style="background-color:#e9edf2;">
<img style="display:block; margin: 0 auto; padding: 20px 0 20px 20px;" alt="Integration 101" src="https://i.octopus.com/blog/2018-03/blogimage-buildserver.png" />
</div>

In my 3 years providing support to our users at Octopus, *"Integrating Octopus with build servers"* is probably the subject I answered the most questions about. In this blog post I'm going to try to give a few tips on how you should approach this task if you are starting it from scratch, regardless of the build server technology you are using.

This blog post is aimed for users that are just starting their journey integrating Octopus into their continuous integration (CI) pipeline. If you already have this up and running, there might not be too much value for you here :)

 !toc

## Separation of Concerns

In a way, Octopus and every build server technology out there can seem pretty similar at a first glance. A few things in common they have are:

- Both tools belong in CI pipelines and are aimed to help development teams getting their code out there faster and more reliably.
- Both are basically process runners. You define a process filled with steps, you execute it and magic happens.

But that's pretty much where the similarities stop. If you want to integrate any 2 tools, you need to have a clear image of which task is each tool going to take care of. 

The below are two (very simplified) lists of what each tool's role should be in an ideal CI pipeline.

### What should the build server do?

- **Compile your binaries**. This means running `MSBuild`,`NPM`,`javac`,`dotnet.exe`, etc, and dropping your compiled app to a folder on the build agent.
- **Run tests**.
- **Pack and Push your app to a repository**. Once the tests passed, create a package with the output of your build and push it to your repository.
- **Call the next tool in the CI pipeline**. In our case we'll be calling Octopus to tell it to create and deploy a release.

### What should Octopus do?

- **Provision infrastructure (optional)**. If you need to create an `Azure WebApp`, an `Elastic Beanstalk` instance, scale up a VM set, create an IIS/Tomcat website or anything that's related to setting up *the place* where you'll be putting your compiled code, adding this task as one of the first steps in your Octopus deployment process is a very good idea. Octopus currently supports [Terraform](https://octopus.com/docs/deploying-applications/terraform-deployments), [AWS CloudFormation](https://octopus.com/docs/deploying-applications/aws-deployments/cloudformation) and [Azure RM Templates](https://octopus.com/docs/deploying-applications/azure-deployments/resource-groups) for this.
- **Set configuration values in your application before deployment**. Ideally, the content of your package should be deployable to *any* environment in your lifecycle, and the only thing that should be different are environment-specific configuration values such as connection strings, passwords/API Keys, etc. Octopus has [a wide set of features](https://octopus.com/docs/deployment-process/configuration-files) to deal with configuration modifications as deployment time that can be used for this.
- **Deploy your Application**.

## So how should I start integrating Octopus into my CI pipeline?

One of the most common mistakes I've seen people make when starting this task was to try to configure too many things at the same time, without fully understanding what each tool brings to the table. In this blogpost I'm not gonna share any details and how to set things up (we have great documentation for that). Instead, I want this blogpost to work as a guideline/checklist that'll help you get this task done in an ordered fashion.

We're gonna split this task into 3 well delimited stages:  **The Build**, **The Deployment** & **The Integration**. Each stage will have its own **End Goal** that we are going to focus on.

:::hint
**Working with a teammate?**
Stages **1** and **2** can we worked on in any order, because they won't be touching each other until we reach stage 3. This means that if you are working with a teammate on this integration, he could be focusing in stage **1** while you focus in stage **2** (you could even bet a beer to see who finishes first).
:::

### Stage 1: The Build

:::success
**End goal:** By the end of this stage, you should be able to run a successful build of your application, and as a result you should have a package of [any of the supported formats](https://octopus.com/docs/packaging-applications/supported-packages) that contains your build output. The package will only be sitting in a folder on your build agent for now.
:::

We are going to split this stage into 2 steps:

#### 1.1) Get your project building successfully

I've seen this step scare many developers, mostly because it forces them to deal with that black box they've been using for a while called "Build Configuration". It's very common in development teams that only 1-2 devs actually know how their build works, and the rest simply click on "Run" and hope for the best.  If you are in the latter group, this might be a good moment to change that situation and pair up with a teammate to learn how your build process works.

To consider this step done, you should be able to get a successful build following these 2 guidelines:

- The build output must be sent to a fixed folder. Every build tool out there has a parameter that allows you to send the output to a directory of your choice. My recommendation is that you send it to a folder called `Build` or `Output` that sits at the root of your build's `WorkDir`.
- The content in that folder should be structured exactly like you expect it to be deployed to its destination. For example if your Website/Pass platform expects a `web.config` and an `index.html` file at the root, then those 2 files should also be at the root of this folder.

**1.2) Get your build output packaged up**

In this step you should be packaging up the contents of the `output folder` mentioned in the previous step into a package of [any of the supported formats](https://octopus.com/docs/packaging-applications/supported-packages). We strongly recommend you to use one of the "Pack application" steps [provided by our plugins](#a-few-words-about-build-server-plugins). If you are not using one of the build servers with a plugin built by our team, you can add a "script" step to your build that runs [Octo.exe pack](https://octopus.com/docs/packaging-applications/creating-packages/nuget-packages/using-octo.exe) to achieve the same.

The only key recommendation here is that you version the package with the same version number of the build that's creating it. So if you are building the project `SupportFlow` and you are running the build `1.0.78`, your package should end up being `SupportFlow.1.0.78.zip`

![Package version == build version](package-version.png)

### Stage 2: The Deployment

:::success
**End goal:** By the end of this stage you should be able to create a release in Octopus and trigger a successful deployment of your application from the command line using [Octo.exe](https://octopus.com/docs/api-and-integration).
:::

We are also going to split this stage into a couple of steps:

**2.1) Upload a test package to the built-in repository**

The idea of this step is simply to get a package with your compiled application on it, so you can use it in the deployment process you are about to setup. If you've already finished **Stage 1**, you can grab one of the packages that was created during your builds. If you haven't finished that stage yet, simply compile your app locally and package the output using [Octo.exe pack](https://octopus.com/docs/packaging-applications/creating-packages/nuget-packages/using-octo.exe).

Once you have the package, push it to the [Octopus built-in repository](https://octopus.com/docs/packaging-applications/package-repositories/pushing-packages-to-the-built-in-repository#PushingpackagestotheBuilt-Inrepository-UsingtheOctopuswebportal) and make sure you can see it in the web portal under `Library -> Packages`

![Package in repository](package-in-repository.png)

You only need 1 package for the next step, which you'll be using over and over until you get the Deployment Process right. If your deployment process will be using more than one package (perhaps deploying a *WebApp* and a *Cloud Service* separately ), repeat this process for each package you'll be needing.

**2.2) Design your deployment process and run it** 

Now here's where you finally start doing things in Octopus. We won't get into too much detail here, because [that's what our documentation is for](https://octopus.com/docs/deploying-applications). But the overall idea of this step is that you setup your Deployment Process in Octopus using the packages you uploaded in `2.1`, and that you are able to run it successfully through the UI.

If this is your first time setting up your Octopus project, this probably is the step where you'll be spending most of your time. At this point don't bother too much about the version number of the packages you are using or the release number in Octopus. The sole purpose of this step is that you get comfortable with your Deployment Process and that you fully understand what each step brings to the table.

So sit back and trigger as many deployments as you need :)


**2.3) Create a release and trigger a deployment using Octo.exe**

In the previous step you learned how to create a release and trigger a deployment from the Web Portal. The goal of this step is that you learn to do the same thing, but using `Octo.exe`.

If you don't know about this CLI tool, the TLDR is that its a command line application that talks to the [Octopus API](https://octopus.com/docs/api-and-integration/api) and helps you do some of the most frequently used actions against your Octopus Instance. You can read about all the functionality it provides in [this document](https://octopus.com/docs/api-and-integration/octo.exe-command-line)

The command you should be paying attention to is [create-release](https://octopus.com/docs/api-and-integration/octo.exe-command-line/creating-releases). A few tips about this command:

- If you use the `--deployTo` parameter, it will not only create the release, but also deploy it to an environment. It basically combines the commands `create-release` and `deploy-release`
- use `--progress` to be able to see the deployment log in the console at it executes. Otherwise the command will only create a task in Octopus, and you'll be forced to go to the Web Portal to see how the deployment went.
- use `--whatIf` to see what would happen if you ran that command, without actually triggering anything in Octopus.

:::hint
Every single build server integration out there (at least the ones built by the Octopus team) is simply a UI wrapper around this CLI tool. So the knowledge you gain from this step will come in really handy on the next stage.
:::

### Stage 3: The Integration

:::success
**End goal**: By the end of this stage, you should be able to add a new step to your build process that triggers a deployment in Octopus.
:::

Now this stage is where we'll put together everything we did in the two previous stages. For this reason its necessary that you finish both of them successfully.

If you are using any of [the build servers mentioned in the below section](#a-few-words-about-build-server-plugins), then we recommend you to install the plugin/extension for it so you can use it in the below steps.

**3.1)Push the package to a repository from the build** 

In `1.2` you created a package from your build, but until that point the package was just sitting on your build agent. In this step you'll need to tweak your build process in order that *after* creating the package, it pushes it to your package repository. 

Depending on whether you are using one of our plugins or not, and if you are using the Octopus built-in repository to store your packages or not, you'll need to use one of the below approaches:

| Using Plugin | Using built-in repository | Recommended Approach |
| :-----------------: | :-----------------: | :-----------------: |
| Yes | Yes | Use the step with the words "Push package" on its name provided by the plugin |
| Yes | No | Use [Nuget.exe push](https://docs.microsoft.com/en-us/nuget/tools/cli-ref-push) |
| No | Yes | Use [Nuget.exe push](https://docs.microsoft.com/en-us/nuget/tools/cli-ref-push) or [Octo.exe push](https://octopus.com/docs/api-and-integration/octo.exe-command-line/pushing-packages)  |

**3.2)Create a Release/Deployment in Octopus from the build**

If you are using one of our build server plugins, look for a step with the words "Create Release" on its name.

If you are not using one of the build servers mentioned below, don't worry! The knowledge you gained in `2.3` should be more than enough for you to be able to add a Powershell/Bash script step to your build process that runs the same `Octo.exe` command that you already used. To do this, you'll need `Octo.exe` sitting on your build agent at build time. You can achieve that using [this NuGet package](https://www.nuget.org/packages/OctopusTools/) or [this Chocolatey package](https://chocolatey.org/packages/octopustools).

:::hint
**Pro Tip**
Every one of the "Create Release" steps provided by the plugins/extensions will have a checkbox called "Show deployment progress" or similar. If you check that box, your build server will keep the connection open with your Octopus deployment, and it will show you the output of it in your build log. Additionally, if your deployment fails, that step in your build process will also fail. This last bit is extremely handy as you'll know by just looking at your build if the deployment in Octopus failed or not.

If you are using a raw `Octo.exe` call, the equivalent of this feature is the `--progress` parameter.
:::

:::warning
If you run into issues with this step, check our [troubleshooting guide](https://octopus.com/docs/api-and-integration/troubleshooting-integrations-with-build-servers) to get some ideas on how to fix it or to learn how to properly ask for help in our forums.
:::

## A few words about Build Server Plugins

If you check our [API and Integration documentation](https://octopus.com/docs/api-and-integration), you'll notice that our team built a few plugins for some of the most popular build servers out there. These plugins extend the functionality of your build server, by adding some custom steps to do things with Octopus, such as triggering deployments and pushing packages. The below list has links to each plugin documentation, along with the list of steps that each plugin provides.

| Build Server | Step Names                        |
| :----------: | :-------------------------------: |
| [VSTS/TFS](https://octopus.com/docs/api-and-integration/tfs-vsts) | "Create Octopus Release","Deploy Octopus Release","Promote Octopus Release","Package Application","Push Package to Octopus" |
| [TeamCity](https://octopus.com/docs/api-and-integration/teamcity) | "Create Release","Deploy Release","Promote Release","Push Package"(also packs) |
| [Bamboo](https://octopus.com/docs/api-and-integration/bamboo)  | "Create Release","Deploy Release","Pack Package","Push Packages" |

Behind the scenes, all these steps really do is provide a UI to then run `Octo.exe` in the background. So if you are familiar with `Octo.exe` already, you'll be able to understand how each of these steps work a lot easier.

---------------

And that's it! I really hope this guide helps you integrate Octopus into your CI pipeline in a more organized fashion. Please keep in mind that