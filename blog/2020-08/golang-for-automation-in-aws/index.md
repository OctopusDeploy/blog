---
title: Golang for Automation in AWS
description: This post provides a step by step guide on how to use Golang, a popular programming language created by Google, to automate AWS.
author: michael.levan@octopus.com
visibility: private
published: 3020-03-09
metaImage: 
bannerImage: 
tags:
 - Product
 - DevOps
 - Engineering
---

When we think of automation, one of the first things that come to mind is code. A few questions that may be asked are:

1. *What programming language is best for automation?*
2. *What's a straight-forward language for a team to learn?*

Golang, by default, is a procedural based language. That means it's primarily based off of writing functions. When you think of automation code, for example, PowerShell or Python, chances are you're writing some function for a script. Because of that, Golang is a natural fit.

In this blog post, you're going to learn how to use Golang for AWS automation, much like you'd use PowerShell or Python for.

## Prerequisites

To follow along with this blog post, you should have the following:

1. A beginner-level knowledge of GoLang.
2. A beginner-level knowledge of AWS.
3. Visual Studio Code (VS Code), which you can download from [here](https://code.visualstudio.com/).
4. Golang installed, which you can find [here](https://golang.org/doc/install).
5. An AWS configuration on localhost. You can do this by installing the AWS CLI and running `aws configure`.
6. An EC2 instance running

## Figuring out What Packages to use

Before any code can actually be run, you'll need to import some packages, also known as libraries. The primary way you can import packages for AWS in Golang is by pointing directly to the [GitHub](https://github.com/) repository where the package(s) exist.

To start adding in Golang code, you'll need a place to save the code. For the purposes of this blog post, you can save a directory on the Desktop and open it in VS Code.

1. Create a new file and call it `main.go` to store the Golang code for AWS.
2. Add in the following code to create the main package and import a list of packages.

```go
package main

import (
	"fmt"
	"os"

	"github.com/aws/aws-sdk-go/aws"
	"github.com/aws/aws-sdk-go/aws/session"
	"github.com/aws/aws-sdk-go/service/ec2"
)
```

The three packages specific to AWS are:

1. [github.com/aws/aws-sdk-go/aws](http://github.com/aws/aws-sdk-go/aws) - Allows you to connect to the AWS package, specifically, to authenticate and specify a region that you want to work with in AWS
2. [github.com/aws/aws-sdk-go/aws/session](http://github.com/aws/aws-sdk-go/aws/session) - Allows you to create a new session to connect to AWS.
3. [github.com/aws/aws-sdk-go/service/ec2](http://github.com/aws/aws-sdk-go/service/ec2) - Allows you to work with specific EC2 service data, like EC2 status, load balancers, public IPs, etc.

With the packages above, you can connect to AWS and start utilizing the Golang EC2 features that are available.

## Setting up a Function

The previous section showed you how to set up the main package and the AWS-specific package that needs to be imported to interact with AWS at the programmatic level. Now that the AWS packages are imported, it's time to start writing the `ListInstances` function, which will list out metadata from a specific instance ID that you can pass in at runtime.

### The AWS Connection

1. Under the `import`, set up a function called `listInstances` with one parameter to be passed in at runtime called `instanceID`. 

```go
func listInstances(instanceID string) {

}
```

  2. In the function, the first piece of code that will be set up is the connection to AWS with an existing AWS configuration on localhost. The variable will be called `awsConnect` and have error handling. The `session` package will be used to call the `aws` package to point to the existing AWS configuration on localhost. Then, you will specify a specific region (feel free to change this if needed).

```go
awsConnect, err := session.NewSession(&aws.Config{
		Region: aws.String("us-east-2")},
	)
```

### Error Handling

1. Next, you'll add in error handling for the `err` that was configured in the variable. It will be a simple output to the screen if an error occurs.

```go
if err != nil {
		fmt.Println(err)
	}
```

### Initiating the AWS Connection

1. Initiating the AWS connection will be done by using the `New()` method found in the `EC2` package and utilizing the `awsConnect` variable that points to the local AWS configure.

```go
ec2sess := ec2.New(awsConnect)
```

### Outputting EC2 Information

To output the EC2 instance metadata, the `ec2` package is used with the `DescribeInstancesInput`method. Within the `DescribeInstancesInput` method, the `instanceID` variable will be passed in (the `instanceID` parameter will be explained more in the next section for the `main` function).

After the Instance ID is passed in, there will be two print statements:

1. Letting the user know the EC2 info is printing
2. Printing out the instance info using the `DescribeInstances` method.

```go
instanceInfo := &ec2.DescribeInstancesInput{
		InstanceIds: []*string{aws.String(instanceID)},
	}
	fmt.Println("Listing EC2 Instance info...")

	fmt.Println(ec2sess.DescribeInstances(instanceInfo))
}
```

## Configuring the Main Function

In the previous section, you configured the primary function that will be doing all of the legwork. In this section, it's time to configure the main function. The main function will do the following:

1. Run the `listInstances` function
2. Set up the `instanceID` variable that is being used as a parameter in the `listInstances` function.

The `instanceID` variable in the main function is utilizing the `os.Args` package so you can pass in values at runtime. This will allow the code to be reusable and not have any hard-coded EC2 instance IDs.

```go
func main() {
	instanceID := os.Args[1]

	listInstances(instanceID)
}
```

## Running the Code

In the three previous sections, you set up all of the code needed to retrieve the EC2 metadata information. The entire program should look like the below:

```go
package main

import (
	"fmt"
	"os"

	"github.com/aws/aws-sdk-go/aws"
	"github.com/aws/aws-sdk-go/aws/session"
	"github.com/aws/aws-sdk-go/service/ec2"
)

func main() {
	instanceID := os.Args[1]

	listInstances(instanceID)
}

func listInstances(instanceID string) {
	awsConnect, err := session.NewSession(&aws.Config{
		Region: aws.String("us-east-2")},
	)
	if err != nil {
		fmt.Println(err)
	}

	ec2sess := ec2.New(awsConnect)

	instanceInfo := &ec2.DescribeInstancesInput{
		InstanceIds: []*string{aws.String(instanceID)},
	}
	fmt.Println("Listing EC2 Instance info...")

	fmt.Println(ec2sess.DescribeInstances(instanceInfo))
}
```

Once the program is correctly written, it's time to run it.

1. Open up a terminal and change directory (`cd`) to where the `main.go` program exists.
2. Run the following command to run the Golang program.

```go
go run main.go instance_id
```

Once the program has completed successfully, you should see an output similar to the screenshot below:

![](images/1.png)

Congrats! You have successfully used Golang to retrieve EC2 information from AWS.

## Conclusion

There are several ways to automate and many different programming languages to do so, but that doesn't mean all programming languages are the best to automate with. When you're scripting for automation, you'll need a programming language that's easily readable, straight-forward, and built for writing small functions.

In this blog post, you took a look at how to write a small function to retrieve metadata on an EC2 instance and how you can automate the tasks in AWS.