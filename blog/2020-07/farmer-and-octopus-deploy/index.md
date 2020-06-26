---
title: "Farmer: Simpler ARM deployments with Octopus Deploy"
description: Learn how to use Farmer to create and deploy ARM templates with Octopus Deploy 
author: mark.harrison@octopus.com
visibility: private
published: 2020-07-31
metaImage: octopus-farmer.png
bannerImage: octopus-farmer.png
tags:
 - DevOps
 - Product
---

![Farmer ARM deployments and Octopus](octopus-farmer.png)

Having worked with Azure for around a year, it’s clear to see why [ARM templates](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/overview) are popular. They provide a declarative model to generate entire environments at the touch of a button. 

However, if like me, you’ve ever tried to author an ARM template file, you might have come across one of my biggest gripes with them; they rely on strings, and can be prone to human error. There’s no compiler to help me out when I have a typo in a template (and there have been plenty of those!).

I've used C# as my primary development language since 2012, however its functional counterpart, F# has some useful features which can help out with my ARM template dilemma. One area in particular that F# excels, is its built-in [type-safety](https://fsharpforfunandprofit.com/posts/correctness-type-checking/).

In this post, I’ll demonstrate the type-safety in F# in action by using [Farmer](https://compositionalit.github.io/farmer/) to generate an Azure WebApp ARM template, and then walk through how you can use its optional deployment capabilities through Octopus to deploy the WebApp to Azure directly.


<h2>In this post</h2>

!toc

## What is Farmer?

The authors of Farmer [says](https://compositionalit.github.io/farmer/about/):

> Farmer is an open source, free to use .NET domain-specific-language (DSL) for rapidly generating non-complex Azure Resource Manager (ARM) templates.

To use Farmer, you create a [Farmer template](https://compositionalit.github.io/farmer/quickstarts/template/). These are .NET Core applications which reference Farmer via a [NuGet package](https://www.nuget.org/packages/Farmer/), and they define your Azure resources that you wish to create.

## Why is Farmer needed?

Rather than repeating whats already there, I’d encourage you to read the [About](https://compositionalit.github.io/farmer/about/) section of the Farmer documentation for more details on the motivations on creating a DSL for ARM templates.

For me, the highlights are:
 - It provides a set of types that you can use to create Azure resources with, eliminating the chances of creating an invalid template as they are strongly-typed.
 - It can generate simple ARM templates in a very concise manner, and optionally deploy them.

## Create the Template

To create a Farmer Template, we first need to create a .NET Core application. You can do this in your IDE of choice, or if you prefer the command line you can use the `dotnet new` command, passing the template of the type of application you require. 

For a Farmer template, it’s not uncommon to create a `console` application like this:

```bash
dotnet new console -lang "F#" -f "netcoreapp3.1"
```

However, for this example, I’ll create a class library as I will be deploying the template from within [Octopus](#Depoy-the-Template) directly. You do this by passing the `classlib` parameter to the command:

```bash
dotnet new classlib -lang "F#" -f "netcoreapp3.1" -n "SimpleAzureWebApp"
```

:::hint

:::

:::success
You can see the complete Farmer Template on [GitHub](https://github.com/OctopusSamples/farmertemplates/src/SimpleAzureWebApp/farmertemplate.fsx).
:::

## Package the Template

## Deploy the Template

- mention f# full framework in use with script step and F# as warning?
- 

## Conclusion

What’s great about this technique is that you can version control your Farmer templates so that the code that defines your infrastructure can live alongside the code that runs on it!
