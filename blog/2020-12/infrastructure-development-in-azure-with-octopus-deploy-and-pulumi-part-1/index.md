---
title: "Infrastructure as code in Azure with Octopus Deploy and Pulumi: Part one"
description:  Infrastructure-as-code and software-defined-infrastructure is shaping the way we think about cloud infrastructure and services. In this blog post, Michael goes into one of the newest ways to define infrastructure as software with Pulumi.
author: michael.levan@octopus.com
visibility: private
published: 2199-10-10 
metaImage: 
bannerImage: 
tags:
 - Automation
 - DevOps
---

[Pulumi](https://www.pulumi.com/) is an infrastructure as code solution that lets you define your infrastructure in a language you're already familiar with, for instance, Go, Python, or JavaScript.

In this blog post, I'll show you how to create an Azure resource group using a Pulumi project written in Go (Golang) and deploy it with Octopus Deploy.

## Prerequisites

To follow along with this blog post, you should have the following:

- A [GitHub account](https://www.github.com).
- At least one Linux deployment target.
- An Azure subscription that you have access to.
- An Azure Account set up in Octopus Deploy.

## Why Pulumi and Octopus Deploy?

Pulumi is a multi-language cloud development platform that lets you use programming languages (Go, C#, Python, TypeScript, JavaScript, F#, VB) to build cloud services. Whether you want to build virtual machines, networks, or serverless implementations, and literally anything else, Pulumi has you covered. 

For each language that Pulumi supports, there is an SDK available that you can use to interact with different cloud services. For example, the Azure SDK will allow you to create a resource group.

With Pulumi you can write the code to create the infrastructure, and then use Octopus to deploy the code.

Octopus Deploy has community steps that you can use to run Pulumi projects on both Windows and Linux deployment targets, which should cover any environment you need to work in.

## Configuring a Pulumi project

Before diving into writing and deploying the code, there are a few preliminary steps you need to take to start working with Pulumi.

### Account Setup

To use Pulumi, you need to set up a free account. There are several different ways to authenticate, including:

- GitHub
- GitLab
- Atlassian
- Email/password
- SSO

### Creating a Pulumi Project

After you have signed in, you need to create a new Project:

1. Click the blue **+ NEW PROJECT** button.

    After you create a new project, you'll see there are a few options.

![](images/newpulumiproject.png)

2. For the cloud, choose **Azure**. For the language, choose **Go**, and then click **NEXT**.

![](images/azureandgo.png)

3. Add the details about your project. You can either keep the defaults or add in custom metadata.

- **Project name**: The name of the project being created.
- **Project description**: The description of the project being created.
- **Stack**: The stack name (dev, prod, etc.)
- **Configuration**: Under configuration, you'll see a few different types that you can use:
    - Public
    - usgovernment
    - german
    - china

For a dev environment, as long as you don't have any regulations, keeping it `public` will be fine. When complete, click **CREATE PROJECT**.

4. Next, you'll be presented with the **STACK** window. The stack window helps you get started for MacOS, Windows, and Linux. The first place to start is to confirm that you have Pulumi installed.

- For Windows, you can use the [Chocolatey](https://chocolatey.org/) package manager to install Pulumi.
- For MacOS, you can use [Homebrew](https://brew.sh/) to install Pulumi.

5. After you install Pulumi, you need to create a new directory where the project will live, and then change directory (`cd`) into the newly created project directory.

`mkdir azure-go-new-resource-group && cd azure-go-new-resource-group`

6. Next, pull the project from Pulumi into the directory you just created. Run the following command and follow the instructions:

`pulumi new azure-go -s AdminTurnedDevOps/azure-go-new-resource-group/dev`

7. Once the project is pulled down from Pulumi, it's time to deploy it:

`pulumi up`

 8. Now you can open up the new project in an editor or IDE.

## Writing code with Pulumi

The Pulumi project is created and availabel on your local machine. You now have everything you need to start interacting with Pulumi using Go.

The project includes:

- A `go.mod` file, which specifies the required packages. 
- `yaml` configurations files that specify the project and the project name. 
- The `main.go` file, which already includes Go code. 

With every Pulumi project, you'll see starter code by default that shows you which SDKs and packages are used.

### Azure example

Instead of using the default code in the `main.go` file, let's create something from scratch.

The first thing you'll have to specify is the package name and the imports. Since the code is from `main`, the package that you're using will be `main` as well.

From the standard library, `fmt` and `log` will be used to print output to the screen. The two Pulumi packages are for the Azure SDK and the Pulumi SDK:

```go
package main

import (
	"fmt"
	"log"

	"github.com/pulumi/pulumi-azure/sdk/v3/go/azure/core"
	"github.com/pulumi/pulumi/sdk/v2/go/pulumi"
)
```

Next, you need to create a resource group function that contains three parameters:

- The context from Pulumi
- Resource group name
- Location

```go
func newResourceGroup(ctx *pulumi.Context, resourceGroupName string, location string) {
    if ctx == nil {
        log.Println("Pulumi CTX is not working as expected... please check issues on the SDK: github.com/pulumi/pulumi/sdk/v2/go/pulumi")
    } else {
        pulumi.Run(func(ctx *pulumi.Context) error {
            resourceGroup, err := core.NewResourceGroup(ctx, resourceGroupName, &core.ResourceGroupArgs{Location: pulumi.String(location)})
            if err != nil {
                log.Println(err)
            }

            fmt.Println(resourceGroup)
            return nil
        })
    }
```

The `if` statement checks to see if ctx is nil, and allows us to see if there's an issue with the SDK around the context.

The `else` statement, includes the `Run()` function which is used to execute the body of the Pulumi program. As you can see in the SDK on [GitHub](https://github.com/pulumi/pulumi/blob/master/sdk/go/pulumi/run.go), it does appear to require an anonymous function, specifically to pass in the context.

The core of the code is in `core.NewResourceGroup`, which creates the resource group. There is also the ability to add in some error handling.

```go
func main() {
	resourceGroupName := "octopuspulumitest"
	location := "eastus"
	ctx := &pulumi.Context{}

	newResourceGroup(ctx, resourceGroupName, location)

}
```
The `main` function executes run the `newResourceGroup()` function and passes in some parameters at runtime. There is also an empty initialization of the Pulumi context as that's one of the parameters needed in `newResourceGroup()` 

The finished code looks like the following code snippet: 

```go
package main

import (
	"fmt"
	"log"

	"github.com/pulumi/pulumi-azure/sdk/v3/go/azure/core"
	"github.com/pulumi/pulumi/sdk/v2/go/pulumi"
)

func main() {
	resourceGroupName := "octopuspulumitest"
	location := "eastus"
	ctx := &pulumi.Context{}

	newResourceGroup(ctx, resourceGroupName, location)

}

func newResourceGroup(ctx *pulumi.Context, resourceGroupName string, location string) {
	if ctx == nil {
		log.Println("Pulumi CTX is not working as expected... please check issues on the SDK: github.com/pulumi/pulumi/sdk/v2/go/pulumi")
	} else {
		pulumi.Run(func(ctx *pulumi.Context) error {
			resourceGroup, err := core.NewResourceGroup(ctx, resourceGroupName, &core.ResourceGroupArgs{Location: pulumi.String(location)})
			if err != nil {
				log.Println(err)
			}

			fmt.Println(resourceGroup)
			return nil
		})
	}
}
```

## Conclusion

There are a lot of initial steps to configuring Pulumi, but as you can see, it's super powerful. You can take a programming language that you enjoy using and create the infrastructure or services you need in an environment.

In [part two](/blog/2020-12/infrastructure-development-in-azure-with-octopus-deploy-and-pulumi-part-2/index.md), I'll show you how to package the Go code and deploy it with Octopus Deploy.
