---
title: "Infrastructure as code in Azure with Octopus Deploy and Pulumi: Part one"
description: Learn how to define infrastructure as software with Pulumi.
author: michael.levan@octopus.com
visibility: public
published: 2020-12-21 
metaImage: blogimage-infrastructure-development-azure-pulumi-1.png
bannerImage: blogimage-infrastructure-development-azure-pulumi-1.png
tags:
 - DevOps
 - Pulumi
---

![Infrastructure as code in Azure with Octopus Deploy and Pulumi: Part one](blogimage-infrastructure-development-azure-pulumi-1.png)

[Pulumi](https://www.pulumi.com/) is an infrastructure as code solution that lets you define your infrastructure in a language you're already familiar with, for instance, Go, Python, or JavaScript.

In this post, I show you how to create an Azure resource group using a Pulumi project written in Go (Golang), and then how to deploy it with Octopus Deploy.

## Prerequisites

To follow along with this post, you should have the following:

- A [GitHub account](https://www.github.com).
- At least one Linux deployment target.
- An Azure subscription.
- An Azure account configured in Octopus.
- A free [Pulumi](https://app.pulumi.com/signup) account.

## Why Pulumi and Octopus Deploy?

Pulumi is a multi-language cloud development platform that lets you use programming languages (Go, C#, Python, TypeScript, JavaScript, F#, VB) to build cloud services. Whether you want to build virtual machines, networks, or serverless implementations, or anything else, Pulumi can help.

For each language that Pulumi supports, there's an SDK available that you can use to interact with different cloud services. For example, you can use the Azure SDK to create a resource group.

With Octopus Deploy you can use community steps to run Pulumi projects on both Windows and Linux servers, which should cover any environment you need to work with.

### Creating a Pulumi Project

1. Sign into Pulumi.
1. Click **+ NEW PROJECT**.
1. Choose **Azure** for the cloud.
1. Choose **Go** for the lanuage and click **NEXT**.
1. Add the details about your project. You can either keep the defaults or add custom metadata.

   - **Project name**: The name of the project being created.
   - **Project description**: The description of the project being created.
   - **Stack**: The stack name (dev, prod, etc.)
   - **Configuration**: For configuration, you'll see a few different types that you can use:
      - Public
      - usgovernment
      - german
      - china

   For a dev environment, as long as you don't have any regulations, keeping it `public` will be fine. When complete, click **CREATE PROJECT**.

6. If you haven't already installed Pulumi locally, you should install it now. You can use [Chocolatey](https://chocolatey.org/) on Windows, or [Homebrew](https://brew.sh/) on MacOs. 
7. Create a new directory for your project, and then change directory (`cd`) into the newly created project directory.
8. Next, pull the project from Pulumi into the directory you just created. Run the following command and follow the instructions:

`pulumi new azure-go -s AdminTurnedDevOps/azure-go-new-resource-group/dev`

9. After you've pulled the project from Pulumi, it's time to deploy it:

`pulumi up`

## Writing code with Pulumi

Now that you've create the Pulumi project and it is available on your local machine, you have everything you need to start interacting with Pulumi using Go.

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

- Context from Pulumi
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

The `else` statement, includes the `Run()` function which is used to execute the body of the Pulumi program. As you can see in the SDK on [GitHub](https://github.com/pulumi/pulumi/blob/master/sdk/go/pulumi/run.go), it requires an anonymous function, specifically to pass in the context.

The core of the code is in `core.NewResourceGroup`, which creates the resource group. There is also the ability to add in some error handling.

```go
func main() {
	resourceGroupName := "octopuspulumitest"
	location := "eastus"
	ctx := &pulumi.Context{}

	newResourceGroup(ctx, resourceGroupName, location)

}
```
The `main` function executes the `newResourceGroup()` function and passes in some parameters at runtime. There is also an empty initialization of the Pulumi context as that's one of the parameters needed in `newResourceGroup()` 

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

There are a lot of initial steps to configuring Pulumi, but as you can see, it's very powerful. You can take a programming language that you enjoy using and create the infrastructure or services you need in an environment.

In [part two](/blog/2020-12/iac-azure-octopus-pulumi-part-2/index.md), I'll show you how to package the Go code and deploy it with Octopus Deploy.
