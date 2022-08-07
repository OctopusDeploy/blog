---
title: Improving delivery of your deployment steps
description: Our new step package framework means we can deliver new steps to your Octopus instance faster and without the need for upgrades. Learn more.
author: shaun.hevey@octopus.com
visibility: public
published: 2022-08-08-1400
metaImage: blogimage-steppackageframeowkr-2022.png
bannerImage: blogimage-steppackageframeowkr-2022.png
bannerImageAlt: Octopus deployment process editor showing pluggable automation steps or components.
isFeatured: false
tags:
- Engineering
- Product
- Step Templates
---

We've been working on a new way to develop the deployment steps you use in Octopus Deploy, and we call this the step package framework. 

In this post, we explore:

- Why we built a new framework
- What the new framework is and our design decisions
- How we tested and iterated the new framework 
- The benefits for users of Octopus Deploy
- How steps are delivered to your Octopus Server instance with our new step package feed

## Why create a new step package framework?

To help make deploying your software anywhere as easy as possible, Octopus Deploy is an opinionated product. Octopus Server works best when it understands the common targets you're deploying to, ranging from an IIS instance on a server to an Amazon ECS cluster in the cloud. 

To help guide you through the process of interacting with these services, we create opinionated steps. For example, Octopus Deploy steps let you:

- Deploy or update an Amazon ECS Service
- Run a script against Azure or Kubernetes
- Create or destroy infrastructure using a Terraform template

The step package framework is an evolution of how we develop these new steps for you to use in Octopus Deploy. 

To understand why a new framework was required, we must first look at the problems we were trying to solve. 

### How we previously created steps for Octopus Deploy

As Octopus Deploy has evolved, step development has gone through several architectural implementations. Originally, steps were developed in the core of Octopus Deploy. This allowed for tight integration between the components that made up a step and the actual process of using those components to perform the defined deployment. 

![Diagram showing a Step UI component and Step Handler component](img-stepinserver-2022.png "width=500")

One advantage of this approach was that the developer creating these steps had all the libraries and dependencies to develop a step available to them at development time. This approach also enabled tight integration with our React front-end.

However, developing in the core of Octopus had several drawbacks, especially as we developed more steps: 

- Steps could only be shipped as part of the whole Octopus Deploy installer. This meant that to try out new functionality, you had to upgrade your entire Octopus instance. As Octopus often sits in the middle of mission-critical infrastructure, upgrade cycles are long and infrequent meaning you might not be able to use new steps.
- Any required changes to an existing step, including bug fixes, had to go through the whole shipping pipeline and required a  full instance upgrade before the issue was resolved. This meant even small fixes could take weeks to be updated, rather than days or hours. 
- Developing a new step required understanding several big components of Octopus and how they are interlinked. This resulted in increased  delivery times and made it hard to build experimental steps.
- Versioning steps independently of Octopus was impossible in any practical sense. For example, if a cloud provider shipped a new major version of their service, we'd have to build many switches inside of the existing step, instead of just shipping a new version of an existing step that could be used alongside the original version.

In a rapidly changing environment where new cloud services pop up frequently, we realized we needed more flexibility in our approach to keep providing a world-class experience. With the number of steps in Octopus growing, the step package framework was built to solve these issues.

## Introducing the step package framework

The step package framework is a collection of TypeScript libraries that support the creation and integration of steps into Octopus Server. It provides a CLI to ensemble step authors to build and package their new steps.

New steps built using the step package framework are now referred to as a step package. Step packages consist of several components: 

- A UI definition describing the UI elements that Octopus Server should render in the process editor
- An input schema and validator
- A Step Executor, which is the code that is executed at deployment time

Unlike in the previous implementation, all of these elements now exist in the step package itself, instead of inside Octopus Server.

These new steps are built against the API that is exposed from the step package framework and not against the interfaces available in Octopus Server, as steps were previously. The step package framework integrates a step package and all of its constituent parts into Octopus Server.

The step package framework does this by contributing 2 components to Octopus Server: 

1. First is a collection of components to acquire, register, and integrate step packages into Octopus Server, so that step packages can be added and configured to a deployment process. 
2. Second is the step bootstrapper. The bootstrapper is invoked at step package execution time, and it provides the runtime configuration to the step package and then execute the executor code defined in the step package.

![Octopus Service step package framework components showing how they work together](img-steppackageframeworkcomponents-2022.png "width=500")

In addition to the new framework, we also needed to create a new mechanism for delivering step packages outside of Octopus Server releases. To do this, we built a globally accessible step package feed to ship the packaged steps. Unlike how NPM or NuGet feeds operate where any authenticated user can publish a package, the step package feed only contains packages developed and published by Octopus Deploy. After a step is written, built, and packaged, it's ready to be shipped to Octopus instances everywhere. We do this by publishing it to the step package feed.

## Design decisions

While designing the step package framework, we set ourselves several constraints to help shape the final design:

- Steps must be easy to write, even for someone with very little knowledge of how Octopus works.
- The complexity of the underlying Octopus Server components must not leak into individual steps' code.
- Steps must be written in a way that is idiomatic for the ecosystem. For example, if we use JavaScript, it has to be written in a way that most JavaScript developers would expect.
- It should be easy to bring an external component or framework into your step.
- The UI code to configure a step should be declarative and abstract so that step authors don't need to know what specific components are rendered or their internal workings.
- Steps must be versioned and distributed out-of-band of Octopus Server
- Step authors should be able to tell with certainty what inputs they receive from Octopus Server.

When it came to our design decisions, we had to make trade-offs on these guiding principles, as they weren’t all compatible. We made the following design decisions using these tradeoffs and constraints as a guide.


### Steps must be written in a language familiar to most programmers

From the start, we wanted to expose the new way of writing steps to the broadest audience possible, so we had to choose a programming language. A few considerations were:

- Whether the language was popular enough that external libraries and APIs could be easily sourced, eliminating system-level languages that are not widely used to build web applications.
- Whether it was expressive enough to represent complex deployment scenarios. While there's a wide choice of powerful scripting languages today, such as Lua and Groovy, they can become hard to manage with growing size and complexity.
- A rich type system that gives step authors sufficient information about incoming data and allows for compile-time code verification.

We found JavaScript to be popular and expressive enough to match the first 2 conditions. With the addition of TypeScript, we could also build an expressive type system that protects step authors from writing invalid code and provides guidance on how to use the step package framework.

Also, JavaScript projects can be rolled up into a single distributable file, simplifying both publishing and local deployments, especially when things are frequently moving across networks (as is the case with what Octopus does).

### Execution environment

Naturally, native JavaScript execution today is synonymous with Node.js. This suits Octopus as it's cross-platform, widely supported, and light enough to quickly launch in any environment. It's also portable, so we can ship the environment and the executable right to the destination (for example, to a deployment target or a Worker) without compatibility concerns.

### Declarative UI

Early on, we explored different options for representing the user interface of a step. One option was leveraging React and letting users write their own components, using React’s mechanism for changing the renderer. 

However, we decided against it for multiple reasons:

- This flexibility leads to a large compatibility surface, meaning that keeping Octopus Server and the numerous steps and their versions in sync would be difficult.
- We could never support the whole range of libraries available in the React ecosystem. Because of that, it would be difficult to say with certainty which libraries are supported, and so writing React code for an Octopus step would become a different experience from writing React code for the web.
- There are implicit assumptions about React code: you can add your own primitive components and write styles for them, but none of these assumptions would work in a constrained environment.

With that in mind, we decided to decouple the mechanics of how the UI in Octopus Server works and how steps define their portion of it. Instead of asking step authors to write their own components on top of our primitives, we let the developers link their inputs with the semantic components they'd like to represent those inputs. For example, here's the code to show a text field:

```typescript
text({
    input: inputs.myTextInput,
    label: "Some text field",
    helpText: "The help text for the field"
})
```

As the code is declarative and not tied to any particular implementation of a text box, we could easily swap it out for something else without step authors having to do any work. 

Another benefit is that we can use the type system to guide developers toward the right component. For example, we can check that only number inputs are used for the number component at build time. 

The declarative style also allows a well-defined contract between Octopus Server and step packages, helping us reduce the compatibility surface and be more deliberate about breaking changes between the framework versions.

### Versioning

Our biggest goal for the new framework was the ability to ship new steps quickly without having to release a new version of Octopus Server. A consequence of achieving that goal is that steps must also have their own mechanism of versioning, upgrades, and migration. It also means Octopus Server must know precisely: 

- Which step versions it supports
- How to migrate from one version of a step to another
- Which steps versions can be migrated silently, and which migrations require additional user input
- What to do with incompatible steps

We solved this by producing granular compatibility manifests on both sides of the equation. Each version of Octopus Server has a set of ranges of internal components that it can guarantee are compatible, and we inject the specific versions of these components into the step package at build time. Then we can easily tell not only if a step package is compatible, but which specific portions of the package are already compatible and which require migration to the latest version.

We also built implicit guarantees into Octopus Server around version numbers, so that step authors can be sure that if they, for example, start building a step for version 2022.1, their step is always going to run on that version no matter how many minor versions and hotfixes we release later on.

### Dogfooding the new framework

At Octopus, we believe the best way to validate product ideas is to use the product yourself in a real environment, known as “dogfooding”.

After we complete our first iteration of the step package framework, it was time to refine and validate our assumptions and design decisions. To do this we built 2 fully featured steps using the new framework: 

- Deploy an ECS Service
- Update an ECS service

The difference between these 2 steps is that the Deploy an ECS Service step builds a new ECS service definition and deploys it as a Cloud Formation template from scratch, whereas the Update an ECS Service step updates an existing set of definitions. 

With both of these new steps, we started from a blank page, just as any other step author would when using the framework in future. This allowed us to test that the decisions that went into the framework’s design were going to hold up. Mostly, they did, but we learned a few lessons that led to further improvements.  

### Framework improvements

#### The need for new UI components

A step UI component is a part of the step package framework that lets the step authors define the user interface of the step. When the ECS project started, the framework supported most of the basic UI components that can be used to construct an interface with the same look and feel as other built-in Octopus steps. However, the team realized some advanced components were needed, such as the container image selector component.

One feature that required a new UI component was the ability to allow exporting step information to a CloudFormation Template. Implementing these new UI components in our declarative UI presented some challenges. As this is a shared set of components for all step packages, we had to consider how to make the new component generic enough so it could be used by other step packages but still support all the functions required for the ECS steps. This challenge is a conscious trade-off built into the framework that significantly reduces the complexity of authoring a new step at the cost of exposing only a subset of Octopus functionality to the steps.

#### The end-to-end testing framework

One critical aspect the team considered during the development of the ECS steps was how to write integration tests. The tests are required to construct all resources needed in the cloud service so that the deployment of the steps being tested can be executed. 

We needed to set up a large number of resources in AWS before the tests could actually run. We decided to have a Terraform template spin up the resources for the integration test to eliminate the need for static resources. 

Realizing these needs would also be applicable to other types of steps in future, such as those using Azure or GCP infrastructure, we developed a testing framework that acquires and sets up infrastructure from a Terraform template supplied by the client. The client declares the required cloud provider (Azure, AWS, or GCP) and provides the credentials. The framework then outputs a random resource code that can both be used in the step authors’ own Terraform templates and in the tests to access and validate resources.

This testing framework facilitated the development of multiple integration tests we implemented in the ECS steps and is an essential tool for step authors working on any steps that require infrastructure setup in major cloud providers.

#### The pain of testing the steps locally

Steps built with the step package framework are independently developed from Octopus Server. So, we were required to manually integrate the under-development step packages to Octopus Server when we wanted to see how the steps worked visually. To do that, we needed to:

- Build and output the zipped step package from the step project
- Copy the zip files to the local folder inside the Octopus Server repository
- Delete any cache files of the previous version of the step
- Restart/rebuild Octopus Server

This process was time consuming during the development of the ECS steps. As a result, we implemented a local feed that allows Octopus Server to periodically fetch step packages locally and load them to a running Octopus Server instance. This enables step authors to conveniently apply their step package changes without restarting their Octopus Server.

## How the step package framework benefits you

The step package framework is built to help Octopus develop steps in a new way, but what does this mean for you as a user of Octopus Deploy?

- We can now build steps independently of Octopus Server, so they're no longer tied to our release pipeline. 
- We can now ship steps outside of an installation or upgrade of Octopus Server, giving you quicker access to steps without having to upgrade. 

:::hint
Please note that while step packages are packaged and delivered separately from Octopus Server, they still need to be installed in Server to use them.
:::

### Delivery of step packages

After a step package is available on the step package feed, the next time an Octopus Server instance checks the feed for new steps it will download the step package and be available in your Octopus Server instance immediately. This is available if you're using Octopus Server 2022.1 or later and your instance has been configured to allow it to connect to our step package feed.
This will happen on a schedule or can be triggered manually. You can find the status of this sync under **Configuration** > **Features** > **Step Template Updates**.

![Step template update configuration screen](step-feed-sync.png "width=500")

### Step package updates

After a step package has been downloaded and processed into an Octopus Server instance, it updates the usages of that step package in your deployment processes depending on the difference between the current and new versions.

Step packages use the semantic versioning format: major.minor.patch. If a new minor or patch version of a step package is released, then your processes will automatically be updated. 

Details on which version of a step package is being used are available in the **About this step** section of the step in your deployment editor.

![Animated GIF showing how to check the version of a step template](step-package-version.gif "width=500")

If a major version is released, then deployment processes using these steps won't be automatically updated to use the new version. This is because major version updates require user intervention to allow future deployments to be successful. To use the new major version, you need to update each deployment process manually to use the new major version. 

![Screenshot showing a step template upgrade available](step-upgrade-available.png "width=500")

If a new major version is available, you'll see the following banner in your step editor. 

![Screenshot showing a step template upgrade successfully](step-upgrade-success.png "width=500")

Upgrading to use the new major version is as easy as clicking the **Upgrade** button and updating your deployment process with the newly required parameters, then clicking **Save**.

## Conclusion

The step package framework is changing how we build and deliver new steps to your Octopus Server instances. It means that new steps can be delivered faster and without the need for upgrades. 

We look forward to bringing you new and improved steps using this framework as we roll out even more functionality. 

Happy deployments!

*Thanks to Ray Nham, Egor Pavlikhin, and Matt Shepherd who all contributed to this blog post.*