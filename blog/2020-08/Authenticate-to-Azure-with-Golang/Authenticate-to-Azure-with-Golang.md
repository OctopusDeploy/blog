---
title: Authenticate to Azure with Golang
description: Authenticating to a cloud platform with each SDK can be extremely different. Learn how to do it today with Go
author: michael.levan@octopus.com
visibility: private
published: 2030-08-12
metaImage:
bannerImage:
tags:
 - DevOps
 - Azure
 - Platform
---

When you're working with any programming language or automation language, chances are, the first thing you'll have to think to yourself is H*ow do I authenticate from the SDK to the cloud platform?* Depending on what SDK you're using, it can differ exponentially. 

For example, PowerShell authenticates via the `Connect-AZAccount` cmdlet, whereas Python can authenticate with app registration or by using the Azure CLI profile. The point is, it's different for each language and/or SDK, so how about [Golang](https://golang.org/) (Go)? If you're a developer in today's world, chances are you've heard of Golang (it's what Terraform, Docker, and Kubernetes is written in). It's one of the fastest-growing and popular languages in the tooling space today.

In this blog post, you'll take a hands-on approach learning how to authenticate to Azure for using the virtual machine client.

## Prerequisites

To follow along with this blog post, you should have:

1. A beginner to intermediate level understanding of Go.
2. An Azure account. If you don't have one, you can sign up for a free 30-day trial [here](https://azure.microsoft.com/en-us/free/).
3. An IDE or script editor, like [GoLand](https://www.jetbrains.com/go/) or [VS Code](https://code.visualstudio.com/).

## Figuring out What Packages to use

The first step for Azure authentication with Go is figuring out what libraries/packages you'll need in the program. For the purposes of this blog post, besides the standard `os` and `fmt` packages, there will be two Azure packages.

To start adding in Golang code, you'll need a place to save the code. For the purposes of this blog post, you can save a directory on the Desktop and open it in VS Code.

1. Open up the text editor/IDE and create a `main.go` in the directory you wish to write code in on the desktop.
2. Add in the following code to start the `main.go` file off with the primary package and what you will be using to connect to Azure.

```go
package main

import (
	"fmt"
	"os"

	"github.com/Azure/azure-sdk-for-go/services/compute/mgmt/2020-06-01/compute"
	"github.com/Azure/go-autorest/autorest/azure/auth"
)
```

- The `[github.com/Azure/azure-sdk-for-go/services/compute/mgmt/2020-06-01/compute](http://github.com/Azure/azure-sdk-for-go/services/compute/mgmt/2020-06-01/compute)` package is to connect to any virtual machine portion of Azure. Whether it's to retrieve virtual machine information, create a virtual machine, or view metadata.
- The `[github.com/Azure/go-autorest/autorest/azure/auth](http://github.com/Azure/go-autorest/autorest/azure/auth)` is the authentication package that this blog post is using to authenticating to Azure from Go.

## Setting up a Function

The previous section dove into what packages you'll need to set up the ability to communicate with Azure at the programmatic level using Go. In this section, you're going to dive into setting up the `AzureAuth` function.

### The Azure Connection

1. Under `import` in the `main.go` file, create the new function as shown below. The function uses an `os.Arg` for `subscriptionID` which you will define in the `main` function later and the `compute.VirtualMachinesClient` type to utilize the authentication method to Azure.

```go
func AzureAuth(subscriptionID string) compute.VirtualMachinesClient {

}
```

  2. Once the new function is created, you can set up the client authentication to Azure using authentication from a local environment.

```go
vmClient := compute.NewVirtualMachinesClient(subscriptionID)
authorizer, err := auth.NewAuthorizerFromEnvironment()
```

### Error Handling and Authentication

The final piece of the code for authentication is going to be a mixed bag of both error handling and authentication. The `if` statement specifies that if `nil` is not null, print out an error, else, let the user know that the authentication is successful and initiate the VM client.

```go
if err != nil {
		fmt.Println(err)
	} else {
		fmt.Println("Auth: Successful")
		vmClient.Authorizer = authorizer
	}

	return vmClient
```

## Configuring the Main Function

Now that the function that's doing the authentication is configured, it's time to configure the `main` function. Due to Go being a procedural-based language, you're going to utilize the `main` function to call the `AzureAuth` function. That way, the function is run in order.

The `main` function is two lines of code:

- Specifying the `os.Arg` for the subscription ID to be passed in at runtime.
- Calling the `AzureAuth` function.
1. Add the following code above the `AzureAuth` function to specify that it's a `main` function.

```go
func main() {
	subscriptionID := os.Args[1]

	AzureAuth(subscriptionID)
}
```

## Running the Code

In the two previous sections, you set up the code that is needed to run the Azure auth program. Below is what the code should fully look like in an editor.

```go
package main

import (
	"fmt"
	"os"

	"github.com/Azure/azure-sdk-for-go/services/compute/mgmt/2020-06-01/compute"
	"github.com/Azure/go-autorest/autorest/azure/auth"
)

func main() {
	subscriptionID := os.Args[1]

	AzureAuth(subscriptionID)
}

func AzureAuth(subscriptionID string) compute.VirtualMachinesClient {
	vmClient := compute.NewVirtualMachinesClient(subscriptionID)

	authorizer, err := auth.NewAuthorizerFromEnvironment()
	if err != nil {
		fmt.Println(err)
	} else {
		fmt.Println("Auth: Successful")
		vmClient.Authorizer = authorizer
	}
	return vmClient
}
```

1. To run the program, you will need an Azure subscription ID to pass in at runtime. Run the following line of code to run the program.

```go
go run main.go your_subscription_id
```

If the authentication was successful, you should see an output similar to the screenshot below.

![](images/1.png)

Congrats! You have successfully authenticated to Azure with Golang.

## Conclusion

Authentication to any platform, whether it's in the cloud or on-prem, is of course the most important first step. Without authentication, the code is essentially rendered useless if you're planning on using it to connect to a platform.

In this blog post, you took a first look at how to authenticate to Azure using Go for virtual machines.