---
title: Authenticate to Azure with Golang
description: Authenticating to a cloud platform with the different SDK can be extremely different. In this post, you learn how to authenticate to Azure with Go.
author: michael.levan@octopus.com
visibility: public
published: 2020-09-01
metaImage: go-auth-with-azure.png
bannerImage: go-auth-with-azure.png
bannerImageAlt: Authenticate to Azure with Golang
tags:
 - DevOps
 - golang
 - Azure
---

![Authenticate to Azure with Golang](go-auth-with-azure.png)

When you’re working with any programming or automation language, chances are, one of the first things you need to figure out is *How do I authenticate from the SDK to the cloud platform?* The process can vary a lot depending on which SDK you’re using. 

For example, PowerShell authenticates via the `Connect-AZAccount` cmdlet, whereas Python can authenticate with app registration or by using the Azure CLI profile. The point is, it’s different for each language and SDK, so how about [Golang (Go)](https://golang.org/)? If you’re a developer in today’s world, you’ve probably heard of Golang (it’s what Terraform, Docker, and Kubernetes are written in), and it’s one of the fastest-growing and popular languages in the tooling space today.

In this blog post, you’ll take a hands-on approach to learn how to authenticate to Azure to use the virtual machine client.

## Prerequisites

To follow along with this blog post, you need:

1. A beginner to intermediate level understanding of Go.
2. An Azure account. If you don’t have one, you can sign up for a [free 30-day trial](https://azure.microsoft.com/en-us/free/).
3. An IDE or script editor, like [GoLand](https://www.jetbrains.com/go/) or [VS Code](https://code.visualstudio.com/).

## Which packages to use

The first step for Azure authentication with Go is figuring out which libraries/packages you need in the program. For the purposes of this post, besides the standard `os` and `fmt` packages, there are two Azure packages.

1. Create a directory on your Desktop and open it in VS Code.
1. Create a `main.go` file in the directory.
1. Add the following code to start the `main.go` file with the standard packages and the packages you need to connect to and authenticate with Azure:

```go
package main

import (
	"fmt"
	"os"

	"github.com/Azure/azure-sdk-for-go/services/compute/mgmt/2020-06-01/compute"
	"github.com/Azure/go-autorest/autorest/azure/auth"
)
```

- [github.com/Azure/azure-sdk-for-go/services/compute/mgmt/2020-06-01/compute](http://github.com/Azure/azure-sdk-for-go/services/compute/mgmt/2020-06-01/compute) is used to connect to any virtual machine on Azure, whether it’s to retrieve virtual machine information, create a virtual machine, or view metadata.
- [github.com/Azure/go-autorest/autorest/azure/auth](http://github.com/Azure/go-autorest/autorest/azure/auth) is the authentication package that this blog post uses to authenticate to Azure from Go.

## The Azure connection

Next, you’re going to set up the `AzureAuth` function:

1. Under `import` in the `main.go` file, create the new function shown below. The function uses an `os.Arg` for `subscriptionID` which you will define in the `main` function later and the `compute.VirtualMachinesClient` type to use the authentication method to Azure:

```go
func AzureAuth(subscriptionID string) compute.VirtualMachinesClient {

}
```

2. After the new function is created, you can set up the client authentication to Azure using authentication from a local environment:

```go
vmClient := compute.NewVirtualMachinesClient(subscriptionID)
authorizer, err := auth.NewAuthorizerFromEnvironment()
```

## Error handling and authentication

The final piece of the code for authentication is a mixed bag of both error handling and authentication. The `if` statement specifies that if `nil` is not null, print out an error, else let the user know the authentication is successful and initiate the VM client:

```go
if err != nil {
		fmt.Println(err)
	} else {
		fmt.Println("Auth: Successful")
		vmClient.Authorizer = authorizer
	}

	return vmClient
```

## Configure the main function

Next, we’ll configure the `main` function. Because Go is a procedural-based language, you’re going to use the `main` function to call the `AzureAuth` function. That way, the function is run in order.

The `main` function is two lines of code that achieves the following:

- Specifies the `os.Arg` for the subscription ID to be passed in at runtime.
- Calls the `AzureAuth` function.

Add the following code above the `AzureAuth` function to specify that it’s a `main` function:

```go
func main() {
	subscriptionID := os.Args[1]

	AzureAuth(subscriptionID)
}
```

The code should now like this in the editor:

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

To run the program, you need an Azure subscription ID to pass in at runtime. Run the following line of code to run the program:

```go
go run main.go your_subscription_id
```

If the authentication was successful, you should see output similar to this: 

![Terminal output showing authentication was successful](images/1.png)

Congrats! You have successfully authenticated to Azure with Golang.

## Conclusion

Authentication to any platform, whether it’s in the cloud or on-premises, is an important first step. Without authentication, the code is essentially rendered useless if you’re planning on using it to connect to a platform.

In this blog post, you took a first look at how to authenticate to Azure using Go for virtual machines.
