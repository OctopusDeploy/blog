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

I’ve used C# as my primary development language since 2012, however its functional counterpart, F# has some useful features which can help out with my ARM template dilemma. One area in particular that F# excels, is its built-in [type-safety](https://fsharpforfunandprofit.com/posts/correctness-type-checking/).

In this post, I’ll demonstrate the type-safety in F# in action by using [Farmer](https://compositionalit.github.io/farmer/) to generate a simple Azure WebApp ARM template, and then walk through how you can use its deployment capabilities through Octopus to deploy different WebApps to Azure directly.


<h2>In this post</h2>

!toc

## What is Farmer?

The authors of Farmer [says](https://compositionalit.github.io/farmer/about/):

> Farmer is an open source, free to use .NET domain-specific-language (DSL) for rapidly generating non-complex Azure Resource Manager (ARM) templates.

To use Farmer, you create a [Farmer Template](https://compositionalit.github.io/farmer/quickstarts/template/). These are .NET Core applications which reference Farmer via a [NuGet package](https://www.nuget.org/packages/Farmer/), and they define your Azure resources that you wish to create.

## Why is Farmer needed?

Rather than repeating whats already there, I’d encourage you to read the [About](https://compositionalit.github.io/farmer/about/) section of the Farmer documentation for more details on the motivations on creating a DSL for ARM templates.

For me, the highlights are:
 - It provides a set of types that you can use to create Azure resources with, eliminating the chances of creating an invalid template as they are strongly-typed.
 - It can generate simple ARM templates in a very concise manner, and optionally deploy them.

## Create the Farmer Template

To create a Farmer Template, we first need to create a .NET Core application. You can do this in your IDE of choice, or if you prefer the command line you can use the `dotnet new` command, passing the template of the type of application you require. 

It’s typical to use a console application for a Farmer Template, and you can create one with the `dotnet new console` command: 

```bash
dotnet new console -lang "F#" -f "netcoreapp3.1" -n "SimpleAzureWebApp"
```

This creates a new F# .NET Core 3.1 application with the name **SimpleAzureWebApp**, using the `-n` parameter we supplied.

Next, we need to add Farmer to the project, by running the `add package` command:

```bash
dotnet add SimpleAzureWebApp package Farmer
```

Now we have our dependencies, we can go ahead and edit the `Program.fs` file which was auto-generated for us when we created the new console application.

**TL;DR**

If you want to see the complete program, skip straight to the [end](#complete-farmer-template) or view the [source code](https://github.com/OctopusSamples/farmertemplates/src/SimpleAzureWebApp/Program.fs). If not, read on!

### Template Parameters

To make the Farmer Template more flexible, we’ll add some parameters to the application. This will allow us to supply different values to our SimpleAzureWebApp application and Farmer will create our resources in Azure based on those values. 

The first three we need are related to authenticating with Azure. These values can be obtained by creating an [Azure Service Principal](https://docs.microsoft.com/en-us/azure/active-directory/develop/app-objects-and-service-principals).

- **AppId** - The application identifier used for the Service Principal.
- **Secret** - The password used for the Service Principal.
- **TenantId** - The ClientId used for the Service Principal.

:::warning
**Securing Credentials:**
You should store credentials used to login into Azure in a secure location, such as a Password Manager, or your Octopus Deploy instance using, preferably, an [Azure Account](https://octopus.com/docs/infrastructure/deployment-targets/azure#azure-service-principal) or [Sensitive variables](https://octopus.com/docs/projects/variables/sensitive-variables). You should also avoid committing them into source control.:::

To run the application, we’ll also supply:

- **Resource Group Name** - Which resource group to add the Azure WebApp to.
- **WebApp Name** - The name to give to the Azure WebApp.
- **WebApp Sku** - What type of [App Service plan](https://azure.microsoft.com/en-us/pricing/details/app-service/) to use for the WebApp.
- **WebApp Location** - Which data center location to host the Azure WebApp in.

To add our required parameters, the code looks like this:

```fs
let azAppId = argv.[0]
let azSecret = argv.[1]
let azTenantId = argv.[2]
let azResourceGroupName = argv.[3]
let azWebAppName = argv.[4]
let azWebAppSku = argv.[5]
let azWebAppLocation = argv.[6]
```
This simply assigns the parameters from the argument collection supplied to the program when it runs, based on their position from the command line.

:::hint
**Parameter validation:**
I don’t show parameter validation in this example, but you’d likely want to consider adding it to your Farmer Template to ensure they have acceptable values.
:::

### Define Azure Resources

After we have our parameter values, we can define our Azure WebApp using F#:

```fs
let webAppSku = WebApp.Sku.FromString(azWebAppSku)
let webApp = webApp {
    name azWebAppName
    sku webAppSku
}
```
Here we assign the WebApp Sku to a variable named `webAppSku`. This is done by a helper function to return a strongly typed `Sku`. Then we create our `webApp` variable using the Farmer [Web App builder](https://compositionalit.github.io/farmer/api-overview/resources/web-app/).

Next, we create our ARM deployment using the Farmer [ARM deployment builder](https://compositionalit.github.io/farmer/api-overview/resources/arm/), which in this example, consists of the location to deploy to, and the Azure WebApp as defined previously:

```fs
let deployLocation = Location.FromString(azWebAppLocation)
let deployment = arm {
    location deployLocation
    add_resource webApp
}
```

#### Built-in Type safety 

In both of the previous code examples, the power of the F# type-system comes into its own. It’s not possible to create a value that is invalid according to its type.

Let’s see an example. Suppose I wanted to create our Azure WebApp with a `Sku` which had a value of `VeryFree`. If I try to create that in our application, the compiler will give me a warning, and it won’t build:

![Farmer Compiler warning](farmer-sku-invalid.png)

This is where Farmer really excels over crafting your own ARM template by hand. Its use of F# provides you with type safety to ensure you have valid templates from the outset.

:::hint
**Farmer and ARM:**
There is a more detailed comparison between Farmer and ARM templates [here](https://compositionalit.github.io/farmer/arm-vs-farmer/).
:::

### Generate ARM Template

Once you have your Azure resources modelled, Farmer supports different ways to [generate the ARM template](https://compositionalit.github.io/farmer/api-overview/template-generation/). One way is to write it out to a file directly:

```fs
printf "Generating ARM template..."
deployment |> Writer.quickWrite "output"
printfn "all done! Template written to output.json"
```
You can then take this file and deploy to Azure using whatever method you prefer.

#### Deployment to Azure

In addition to generating the ARM template, you can also, optionally, have Farmer execute the deployment to Azure when the application runs.

:::hint
**Azure CLI required**
If you use the Integrated deployment to Azure feature, you will need the Azure CLI installed on the machine where you run your application.
:::

In our example **SimpleAzureWebApp** application, we’ll take advantage of this feature. 

Before we can execute the deployment, we need to authenticate with Azure. Farmer comes with a `Deploy.authenticate` command, and you call it passing in your credentials supplied to the application previously, like this:

```fs
Deploy.authenticate azAppId azSecret azTenantId
|> ignore
```
If there are any errors authenticating, an error will be raised. If the login succeeds, we then need to get Farmer to execute our deployment, using the `Deploy.execute` command:

```fs
deployment
|> Deploy.execute azResourceGroupName Deploy.NoParameters
|> ignore
```
Just like the authentication, any errors on deployment will be surfaced as an error.

### Complete Farmer Template
And that’s all there is to our application. Here is the finished `Program.fs` file:

```fsharp
open Farmer
open Farmer.Builders
open SimpleAzureWebApp.SkuExtension

[<EntryPoint>]
let main argv =
    
    let azAppId = argv.[0]
    let azSecret = argv.[1]
    let azTenantId = argv.[2]
    let azResourceGroupName = argv.[3]
    let azWebAppName = argv.[4]
    let azWebAppSku = argv.[5]
    let azWebAppLocation = argv.[6]

    let webAppSku = WebApp.Sku.FromString(azWebAppSku)
    let webApp = webApp {
        name azWebAppName
        sku webAppSku
    }

    let deployLocation = Location.FromString(azWebAppLocation)
    let deployment = arm {
        location deployLocation
        add_resource webApp
    }

    printf "Authenticating with Azure\n"
    Deploy.authenticate azAppId azSecret azTenantId
    |> ignore

    printf "Deploying Azure WebApp %s (%s) into %s using Farmer\n" azWebAppName azResourceGroupName azWebAppLocation
    
    deployment
    |> Deploy.execute azResourceGroupName Deploy.NoParameters
    |> ignore

    printf "Deployment of Azure WebApp %s (%s) complete!\n" azWebAppName azResourceGroupName

    0 // return an integer exit code
```


## Package the Farmer Template

Use CLI / build server

plug for guides

## Deploy the Farmer Template

- mention f# full framework in use with script step and F# as warning?
- 

## Conclusion

What’s great about this technique is that you can version control your Farmer Templates so that the code that defines your infrastructure can live alongside the code that runs on it!
