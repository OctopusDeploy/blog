---
title: The Octopus Deploy Terraform Provider
description: Learn all about the new beta Octopus Deploy Terraform provider
author: michael.levan@octopus.com
visibility: private
published: 2030-09-10
metaImage:
bannerImage:
tags:
 - DevOps
 - Terraform
---


Infrastructure-as-code ([IaC](https://searchitoperations.techtarget.com/definition/Infrastructure-as-Code-IAC#:~:text=Infrastructure%20as%20code%2C%20also%20referred,hardware%20devices%20and%20operating%20systems.)) has made it possible to move past *manually clicking a bunch of buttons to create infrastructure resources*.

Terraform is quickly becoming the go to IaC solution. Terraform is an open-source IaC solution that allows you to define your infrastructure using a fast growing functional-based programming language called Hashicorp Configuration Language (HCL).

In this blog post, you get a first look at the Octopus Deploy Terraform provider and what it has to offer.

## Why IaC?

The further the world goes down the cloud engineering and development route, the clearer it is that we cannot continue creating infrastructure manually. Not only does it take a long time, but it's prone to failures. There's no one person in the world that can say they've never done *something* by mistake when creating a resource manually. Whether it was something as small as a typo, as humans, we're prone to mistakes.

When it comes to IaC, mistakes can still be made in code, but they're much less. When you're writing code and storing it in source control, you can do things to prevent mistakes as much as possible such as:

- Code reviews
- Automated testing to ensure the code works as expected
- A secure place to store the desired state
- A place where everyone can see what's being worked on and collaboration can take place

## What is a Terraform Provider?

A provider is what you interact with when writing HCL code. For example, there's a provider for Azure. The Azure provider gives you the ability to write HCL code to interact with Azure at the code level. That's exactly what the Octopus Deploy provider does as well.

The provider communicates with the client via calls based on packages, types, and models. The provider is what you write to actually write what Terraform resources you're going to use.

For example, let's say you want to create a new project group in Octopus Deploy. There's a resource inside of the Terraform provider that's used to create a project that. That resource in the Terraform provider makes a call to the `ProjectGroup` type, which then calls a function to create a new project group.

![](images/1.png)

## What can the provider do?

At this point you have a bit of background on the client, the provider, and why you would want to use IaC, but how about the actions that the provider can actually do?

The provider can create, update, replace, or delete anything until you're at the project level. This includes:

- AWS accounts
- Azure accounts
- UsernamePassword accounts
- SSH acounts
- Certificates
- Channels
- Environments
- Feeds
- Library sets
- Lifecycles
- Feeds
- Project groups
- Projects
- Deployment targets

Keep in mind that the Terraform provider is still in beta, and we're still adding functionality to it.

The only thing that the provider cannot do is, for example, create the underlying infrastructure in say, Azure or AWS. That's because you would need to use the Azure or AWS provider to, for example, create a virtual machine/EC2 instance.

### The backend

First and foremost, what is a Terraform provider even made of? It's most likely not a question you typically ask yourself. When you're writing a Terraform application, you're writing in HCL, but what is the provider itself made out of?

All backend code for Terraform is written in Golang (Go), a procedural based programming language created by Google. Go makes up some of the most popular tools and platforms today including:

- Terraform
- Docker
- Kubernetes

When you're thinking about the backend, there are two parts - the client and the provider.

## Terraform SDK 2.0

When the journey first started to build the Octopus Deploy Terraform provider, the Terraform SDK was on version 1.0. As we were building up the Terraform provider for beta, we noticed that version 2.0 of the Terraform SDK was out.

The Terraform SDK gives us a few key capabilities:

- The `schema` needed to create/read/update/delete a resource
- Schema validation
- Types to specify (strings, ints, etc.)
- Resource datasets that give us new functions to use for things like `Get()` on names and IDs, setting names, setting IDs, etc..

Essentially, anything that you can do in the Terraform HCL (create resources, update resources, etc.) is because of the `schema`.

Because the schema is so important, it made sense to upgrade to the newest version of the Terraform SDK.

2.0 has a lot of features and deprecations of old functionality, but the biggest implementation it gave us personally is better diagnostics. In Go, there is a default `error` type that you can use to return errors for functions. With the 2.0 SDK, there are diagnostics built-in by default, which we really wanted to take advantage of.

We're all about testing and ensuring code quality at Octopus Deploy, and with the Terraform provider, it's no different. Because of that, implementing a new SDK for better diagnostics was a no-brainer.

### The Client

The client itself is an SDK that makes API calls to the platform that you're working in, in this case, Octopus Deploy. The client is the way you're able with the platform at a client/service level. 

You can find the client SDK [here](https://github.com/OctopusDeploy/go-octopusdeploy).

## Testing

We care about testing, like....a lot. The only way anyone can know if code is supposed to do what it's actually supposed to do is by testing. Not only at the moment of writing it, but 3 or 6 or 12 months down the line when new implementations are added. 

The Terraform provider is a moving target, and it should be. With all of the constant implementations of Terraform and HCL, for the provider to stay useful, it needs to change as Terraform changes. Because of that, automated testing for almost every bit of the code is **key**.

We first started off with implementing unit tests for each function in not only the Terraform provider, but the client SDK (go-octopusdeploy). The idea behind so many unit tests was, no matter what we changed (even upgrading to a new Terraform SDK), we would know what broke right away. *Fail fast* is a good way to think about it.

After that, we started thinking about mock tests with Terratest by Gruntwork. Terratest allows you to unit test and mock test the Terraform HCL code. By doing this, we could write Terraform examples and confirm that they actually work. Taking it a step further, after we confirmed they worked, we could continue to confirm they worked with the new implementations to the client SDK and the provider. No matter what changed, we could always run our tests on the examples.

Once we were comfortable with the unit and mock tests implemented, we started thinking about all of the edge cases. We used a practice called Defense In-Depth, which essentially means *test everything, twice. Even if it's not yours*. What I mean by that is, we went as far as testing the SDKs and packages we were using in our code. Even though they weren't written by us, we still wanted to test them to ensure they were doing what they were supposed to be doing. Remember, *fail fast*.

Once the standard tests were complete, we implemented linting. We used a package called `golangci-lint`, which takes **all** of the popular linters (gosec, staticcheck, gofmt, etc.) and ties them into one linter (golangci-lint). 

Finally, after all of the tests were written, we needed a way to run them automatically. This is where the power of GitHub Actions Workflows came into play. We set up multiple workflows for unit testing, linting, integration testing, and mock testing. We took it a step further and for each test, we spun up an instance of Octopus Deploy in a Docker container for each time we ran the GitHub Actions workflow so we could literally test the code against a **real** environment in real-time.

### Contributing

If you would like to contribute, you absolutely can! It's 100% open source and we're welcoming all contributions via pull requests.

## How is this different to Config as Code?

A big push this year for Octopus Deploy has of course been config as code, so what happens to that since there is a Terraform provider? Well, nothing. Config as Code is still 100% needed. The Terraform provider handles everything before getting to the project.

Config as Code picks up where the Terraform provider leaves off and gives you the ability to manage the Project with code. 

The idea of the Terraform provider is to fill the part of the automated deployments before reaching the project.

## Conclusion

Infrastructure-as-code using Terraform and the power of a fully functional-based programming language under the hood is an ideal scenario for software-defined infrastructure. It gives you the ability to not only automate the work you're doing, but ensure quality and value for everyone using the code.