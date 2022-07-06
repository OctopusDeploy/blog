---
title: Microservices and frameworks
description: An overview of microservices and the different frameworks you can use to support them.
author: andrew.corrigan@octopus.com
visibility: private
published: 32022-07-11-1400
metaImage: blogimage-microservicesframeworks-2022.jpg
bannerImage: blogimage-microservicesframeworks-2022.jpg
bannerImageAlt: 3 people building an unstable looking tower with blue blocks, beside 2 people building a stable, lower tower with blue blocks.
isFeatured: false
tags: 
  - DevOps
  - Containers
---

Delivering software through microservices is a modern development approach with many benefits, for developers and customers alike.

Using microservices means developing an application in separate, independent services. Users then access the whole application and its features through a front-end, unaware of the difference in delivery.

Despite the benefits, there are things to consider before developing in microservices, such as the software's structure and the team's processes. Thankfully, many microservices frameworks exist to help lift some of that load. 

In this post, we look at:

- Benefits of developing in microservices
- The concept of frameworks and why you should consider them
- Programming languages and the frameworks that suit them

## Benefits of microservices

Let's imagine you're building a retail website with microservices. You might decide to develop the following features as separate services:

- Website front-end (HTML, CSS, etc)
- Search
- Navigation
- Product database
- Call to actions
- Carousel
- Website widgets (new, popular, or sale items, for example)
- Customer product reviews
- Checkout and payment systems
- Support live chat

Compared to a traditional website delivered as one object, microservices offer the following benefits:

- Product reliability - A problem with one component is less likely to impact your business. For example, people can still search and buy things if your customer reviews component goes down.
- Quicker fixes - If the container with your checkout system becomes corrupt, you can replace it with a fresh image in minutes.
- Reusable components - Opening a second retail website and want both to share the same live chat for support? Both sites can share the same feature.
- Scalability - You can easily scale your website's resources to meet each component's traffic demands.

There are cost and network traffic trade-offs to consider, however. Terence Wong explores these in more depth in his post [Monoliths versus microservices](https://octopus.com/blog/monoliths-vs-microservices).

## Microservices frameworks and why you should consider them

If you want to deliver your product through microservices, there are many ways to plot out how it'll work.

You could start from scratch and feel out a simple product structure along the way with nothing but your code and some containers.

This approach is likely fine for small projects with few features, but things can get complex quickly should you need to scale suddenly. You could see a bunch of unintended impacts, such as:

- The product being hard to troubleshoot  
- Risk of new team members struggling to figure out how things fit together
- Finding dependencies you didn't know you had

This is where adopting an established microservices framework from the start makes things much easier.

As the name implies, a framework is a ready-made architectural structure for software development. A framework: 

- Helps form the shape of your software as it lives on your infrastructure
- Offers teams clarity and focus
- Has tools to help with development
- Helps everyone in your team pull in the same direction.

Best of all, a framework saves you time in planning, development, and support. After all, frameworks are proven, well-worn paths to software delivery. Why spend time making mistakes or plotting structures when someone has already done that for you?

There are countless frameworks available for projects delivered in microservices. Those you consider will depend on your project and the programming language you use.

## Popular frameworks for microservices

Let's look at a handful of popular framework options for different development languages. This is not a comprehensive list, however. There are plenty more frameworks on the market, and we recommend doing your research before committing to one.

All these frameworks are open-source.

### Node.js and JavaScript

#### Molecular

The [Molecular](https://moleculer.services/) microservices framework promises:

- High-speed performance
- Extensible through existing or self-developed plugins
- Fault tolerance through a built-in load balancer, circuit breakers, and more
- Compatibility with popular logging services

#### Koa

[Koa](https://koajs.com/) is a microservices framework that claims to be leaner and more customizable than its predecessor, [Express](http://expressjs.com/). It also aims to make server creation easier.

#### Loopback 4

[Loopback 4](https://loopback.io/doc/en/lb4/) is a microservices framework that includes:

- OpenAPI Spec Driven REST APIs
- Dependency injection through components, mixins, and repositories
- GraphQL support

### Java

#### Micronaut

Micronaut (https://micronaut.io/) is a solid option because of its compatibility with Java-like languages, such as Groovy and Kotlin.

It offers:

- Built-in cloud support
- Easy unit tests
- Quick config
- Support for API services like OpenAPI and Swagger

#### Axon Framework

A microservices framework by [AxonIQ](https://www.axoniq.io/axoniq-products) that's suitable for development using:

- Event sourcing
- [Command Query Responsibility Segregation (CQRS)](https://en.wikipedia.org/wiki/Command%E2%80%93query_separation#Command_Query_Responsibility_Segregation)
- [Domain-driven design (DDD)](https://en.wikipedia.org/wiki/Domain-driven_design)

#### Spring Cloud Functions

A microservices framework from Java-framework provider, Spring.

[Spring Cloud Functions](https://spring.io/microservices) boasts a framework that makes it easy to start small and grow.

Its [Spring Cloud](https://spring.io/cloud) feature helps with connectivity with many service registries, and also offers an API gateway into your project.

They also offer optional metrics through [Micrometer](https://micrometer.io/).

#### Quarkus

Specifically for Java developers who want to work with Kubernetes, [Quarkus](https://quarkus.io/) is a microservices framework meant for use with [GraalVM](https://www.graalvm.org/) and [HotSpot](https://www.oracle.com/java/technologies/javase/vmoptions-jsp.html).

### Python

#### Flask

[Flask](https://flask.palletsprojects.com/en/2.1.x/) is a minimalistic but powerful microservices framework for Python-developed projects.

#### Falcon

[Falcon](https://falconframework.org/) is a microservices framework that plays well with other Python frameworks.

It is:

- Extensible - see some examples on the [Falcon wiki](https://github.com/falconry/falcon/wiki)
- Compatible with both the [Web Server Gateway Interface (WSGI)](https://wsgi.readthedocs.io/en/latest/) and [Asynchronous Server Gateway Interface (ASGI)](https://asgi.readthedocs.io/en/latest/)

#### CherryPy

As another lightweight Python offering, [CherryPy](https://cherrypy.dev/) claims to be as easy as its name.

It is:

- HTTP and WSGI compliant
- Runs on multiple ports
- Extensible through plugins

### Go

#### Go Micro

[Go Micro](https://asim.github.io/go-micro/) promises:

- In-built authentication
- Dynamics configs
- Service discovery through DNS
- Load balancing
- Message encoding
- Async and event streaming

#### Echo

[Echo](https://echo.labstack.com/) promises:

- HTTP routing with zero dynamic memory allocation
- Scalability
- Auto TLS certificates with Let's Encrypt
- Templates
- Data binding and rendering

#### Fiber

Its developers describe [Fiber](https://gofiber.io/) as a Go equivalent of the Express framework. Built on top of Fasthttp, it offers easy routing definitions and helps deliver static files.

### .NET and C#

#### ASP.NET

[ASP.NET](https://dotnet.microsoft.com/en-us/apps/aspnet) is Microsoft's framework for building web apps and services. ASP.NET is especially useful for microservices thanks to its support of Docker images and the ease that you can create APIs for each service.

## What's next?

In this post we looked at microservices benefits, explored why using a microservices framework is a good idea, and listed some popular framework options. 

We also have posts about containerization on the way, including:

- Registries you should consider
- More detailed looks at containerization's benefits
- A deep dive into cloud orchestration and cloud automation
- A look at 'everything as code'

Happy deployments!