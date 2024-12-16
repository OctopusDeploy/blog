---
title: How to choose the tools to run your Kubernetes pipeline
description: This post is an excerpt from our free guide, Kubernetes delivery unlocked, which examines how to set up your Kubernetes pipeline for success.
author: liam.mackie@octopus.com
visibility: public
published: 2025-01-15-1400
metaImage: blog-kubernetes-unlocked-choosing-tools-750x400-x1.png
bannerImage: blog-kubernetes-unlocked-choosing-tools-750x400-x1.png
bannerImageAlt: Illustration of a toolbox containing Kubernetes logos, surrounded by icons representing software tools, interconnected on an isometric grid.
isFeatured: false
tags: 
  - DevOps
  - Kubernetes
---

This blog post is an excerpt from our free guide, [Kubernetes delivery unlocked](https://octopus.com/whitepapers/kubernetes-delivery-unlocked), which examines how to set up your Kubernetes pipeline for success by looking at the different options available throughout the CI/CD process phases.

## Setting up your Kubernetes pipeline

Recommending a single tool for Kubernetes CI/CD is the same as recommending a hammer to build a house. While you could paint with a hammer, it's a lot easier to use a brush. Likewise, you can use a Bash script to build and deploy to Kubernetes, but you'll find it's much harder to get the results you want, no matter how good you are with Bash.

CI and CD tooling are often lumped together, but the 2 phases are very different. CI tooling excels at providing a free-form, pipeline-style interface. The start of the pipeline is code, and the end is a releasable artifact. For Kubernetes deployments, this is often a container and accompanying configuration, like a Helm chart. With CI tooling, it's hard to model the promotion of releases between environments or handle enterprise governance, risk, and compliance needs.

Often, we see teams create custom solutions for Continuous Delivery processes. Rather than investing time and money in a tool, someone on the team codes a quick solution that solves the initial problem. When the next problem arrives, they might add to that solution or devise a new solution. Eventually, the team unwittingly creates a cobbled-together, end-to-end deployment tool, effectively shadow CD. This can turn into a sizable internal engineering team dedicated to building tooling to make it work at scale. Or, more often, only a single person who understands how all the glue hangs together. This adds complexity across your deployment pipelines, which slows down deployment frequency, increases risk, and reduces developer experience.

Solutions like these are not designed for CD. The best CD tools are more opinionated and have features allowing you to exercise your pipeline across multiple environments. It includes tooling to easily customize the configuration to each environment. The key to good CI tooling is having the flexibility to build and test a wide range of code into various artifacts.

It's important to choose the right tooling across your pipeline. To help you decide, we'll walk through the CI/CD process phases. This is a subset of the DevOps infinity loop:

- Building and testing
- Release
- Deploy

## Building and testing

When considering the tooling to build and test your code, you should remember that the goal of CI is to reduce the feedback loop time. This makes you more confident in your code and reduces the time it takes to ship changes - a key metric in developer productivity.

The build and tests should be deterministic, with as many as possible being fast enough to run on a developer machine. After the code is pushed, automated tests in a CI pipeline should run more comprehensive tests, including integration and/or end-to-end tests. Keeping your feedback loop lengths tailored to the different stages of development is key to catching problems early, and, ultimately, being able to ship more changes, faster.

Luckily, most version control services now include CI tools. They include:

- GitHub: GitHub Actions
- GitLab: GitLab CI/CD
- Atlassian BitBucket: BitBucket Pipelines

Standalone tools aim to provide more functionality out of the box, like test history, automatic test parallelization, and other features. Some of these include:

- JetBrains TeamCity
- Codefresh CI
- Jenkins
- CircleCI
- Semaphore

Regardless of the CI system you choose, tests should run quickly and provide fast, easy-to-access feedback to developers. If tests fail, you should surface this quickly to developers. You can do this by running tests automatically in the CI system and providing feedback to pull/merge requests directly. This lets developers immediately know when something has gone wrong, from where they look to when they want to integrate their code.

One of the key problems to manage at scale is test failures. When tests fail because someone's changed the code and broken functionality, everyone is happy with them. When tests fail without any changes to the code or infrastructure that runs them, developers will curse and ignore them. These so-called "flaky tests" generally occur when testing functionality interacting with shared systems, like API calls to an external service or writing to the disk. Keeping track of the tests which frequently fail without changes is invaluable.

Many CI systems integrate ways to keep track of test failures, and flag tests that have been failing more frequently. When choosing a CI system, remember that even if you don't mind if a test fails occasionally now, when thousands of tests are running, these flaky tests can cause delays, and even cause developers to overlook the bugs the tests were originally written to catch.

## Release

After you build and test your code, you want to version and release it. There are many language-specific implementations of version control, which we won't go into. For now, the first thing you need to do is generate a version number. The de facto industry standard for versions is SemVer. SemVer stands for Semantic Versioning, a standard that allows a version to indicate the level of changes made to a piece of software. SemVer uses the convention of major.minor.patch. For example, 1.2.3 indicates a major version of 1, a minor version of 2, and a patch version of 3. These versions are incremented based on the type of change you're making:

- Major: Backwards incompatible change
- Minor: Backwards compatible feature
- Patch: Backwards compatible bug fix

When creating a pre-release version of your software, generally from a feature branch, you must append a pre-release tag to the end of the minor version. This is usually a truncated branch name and other identifying number (like a build number, or commit hash). For example, 1.0.1-new-feature-bafc1c0. You can find more detailed information on the [SemVer website](https://semver.org/), which describes the standard exhaustively.

Where SemVer shines is the amount of tooling available to generate these versions for you. Regardless of the language you're building or the CI tool you're using, there's likely to be a tool to generate a unique version for you.

After you assign a version for the build, and you've run the build and tests in your CI system, you may be asking how to package your code. Previously, you may have provided a binary file in a zip or a tar.gz file. For Kubernetes, your executable should instead be packaged in a container. Building a container is usually done with a Docker file.

An example of building a container for a small go application is below:

```
FROM golang:1.23
WORKDIR /app
COPY hello-world ./
CMD ["/hello-world"]
```

This Docker file uses the base `golang:1.23` image as its base, sets `/app` as the base path, copies in the `hello-world` binary that was built in the CI pipeline, and indicates that when the container starts, it should run `/hello-world`. Your CI system can take this file as an input and generate a container as an artifact. All CI systems should provide documentation on how to build a container.

After you've built the container, you need to store it somewhere for Kubernetes to download and run. You do this by pushing the container to a registry. Docker Hub is the first choice for many organizations, but most CI systems have container registries built-in, like GitHub Container Registry or GitLab Container Registry. If you host your Kubernetes cluster with a cloud-hosting provider, they also likely have a container registry available for you to use. These options generally have more liberal rate limits than Docker Hub.

## Deploy

Deploying is the all-important step in getting your code to your customers. This involves collating the artifacts generated by the build and test steps, configuring your software, and ensuring that it's running in the right place and accessible by your users.

There are many tools and patterns available for you to use to deploy with, but it's worth taking the time to map out the requirements of your deployment process before you choose a tool.

As an example, if you have a small development team, and less than 15 instances of your application, you may find that specialized CD tooling is a hindrance, and you can just deploy using your CI tooling. Alternatively, if you've got a large development team, many dynamic environments, and multiple production environments, you'll find that CD tooling provides many benefits.

### Collating your dependencies

Software is rarely deployed in isolation. One of the first deployment exercises is collating all the dependencies you need for your application to run. Databases, caching systems, file storage, and even specific networking requirements are all things you should consider and note down. This helps build a model of the different things you must deploy. These are likely to change frequently as your software evolves, so try to only focus on the dependencies you use right now.

Before you consider how to manage these dependencies and deploy them automatically, list all the dependencies your software needs. Separate the list into things that can be provided by Kubernetes - like storage and networking, and things provided by other software, like databases.

Along with these external dependencies, you also need to consider the configuration your application needs to run. Separate the configuration by how you apply it to your application. As an example, you might have environment variables and a configuration file. Note down what needs to change under what circumstances. For example, you need to change a configuration variable pointing to your database if you're running a different instance of your
application.

### Define your environments

With your list of dependencies and configuration in hand, consider the type of environment you'll be deploying to. Each type of environment should have consistent configurations, limitations on what can be deployed there, and access policies. Most applications have 3 main environment types:

- Development
- Pre-production
- Production

Development environments are often dynamically created on shared infrastructure and removed when they're no longer required. They let developers run and debug their code in environments outside their local development machine. Most of the time, these environments have stand-in services for dependencies. For example, in pre-production and production environments, you may use a database service from a cloud provider. You can host the database locally in a development environment with fewer resources and availability. Development environments can run branch versions of your application, and developers can usually access the infrastructure of these environments easily.

Pre-production environments can have many names, but they all share a common purpose. They're the step between development and production. Pre-production environments should mirror production environments since they're used to ensure that the application and the deployment process will successfully run in production. These environments should only run code from the main branch, to prevent changes that won't be pushed to your production environment. Changes to the pre-production environment should be limited to only the CD pipeline to ensure that you can consistently push changes to production environments too.

Production environments are where your application ends up serving customers. This environment is by far the most important. At this stage, the code has been automatically tested and deployed multiple times. Since the pipeline itself (including any changes to it) has also been tested in the pre-production environment, you'll have confidence that when you go to deploy to production, everything will work as planned.

After you have a list of environments, consider what configuration or dependency is unique to each. For example, you may have additional logging enabled in development environments or more aggressive caching enabled in production environments. Split the configuration changes into sets, where each environment type has a base set of configuration values.

### Tenants

A tenant refers to a distinct user group with separate data, settings, and configuration. When your software supports these groups of users to use the same instance of your application, the software is called "multi-tenanted". Consider it like housing, where a multi-tenanted application is an apartment building. Multiple tenants share a building (the application instance), which has
shared utilities (dependencies). Despite this, the tenants still have their own apartment (data). When your software isn't built to enable multiple tenants, it's more similar to a house. You need more houses (application instances) to support the same number of tenants, each with its own utility connections (dependencies). 

Even if your software is multi-tenanted, you may have multiple instances of each environment running simultaneously. This is likely for development and pre-production environments, where you may want to test alternate configurations or new features. Separate production environments usually stem from business requirements. You may have a new, large customer demanding an environment closer to their head office, or you may have a requirement to have a separate environment for a bank or government customer.

Consider what needs to change in your configuration for different tenants in each environment. Even if you don't need multiple tenants right now, separating the environment configuration requirements from the tenant configuration requirements is vital so you can modify your deployment tooling easily to create new environments in the future.

### Genericize your configuration

After you understand the parts of your configuration you need to change and the circumstances for those changes, you can make the configuration generic. This is the process of making it easy for you to apply changes in an automated fashion. For example, you may have previously had your app running on your local machine, with a configuration file you manually edited to tweak the options. If you hand-craft a configuration file for every environment, it will get old fast. 

The goal of genericizing your configuration in Kubernetes is to get a set of manifests that let your application run with the least effort. Even though manifests are often simply YAML files, if you need to change an option in 10 different places when you update it, it's only a matter of time until you forget to change it somewhere and break something. There are many templating tools in the Kubernetes ecosystem, including Helm and Kustomize. These powerful tools can help you template your various Kubernetes manifests reliably.

Though there is tooling to help you template your Kubernetes manifests, many CD tools let you directly replace variables inside Kubernetes YAML files. For example, in Octopus Deploy, if you provide a YAML file that uses the `{{Octopus.Environment.Name}}` variable, it gets replaced with the environment you're deploying to. This can be a lower-effort alternative to templating engines like Helm and Kustomize.

### Environment lifecycles

Environment lifecycles are a way to define the stages of your release as it moves through the environments it's destined for. Part of this definition is the environments that the release will be deployed to, but the most important parts are the requirements for the release to progress. This might only be a successful deployment to the previous environment, but you could also require manual approval. As an example, you may want to have a manual test and promotion step when a release transitions between a UAT (user acceptance testing) environment and production.

You can have multiple lifecycles that apply to a category of release. For example, a dev build of your software shouldn't ever go to production, even if it does deploy correctly in pre-production.

The purpose of these lifecycles is to help you define what a blocking problem is for any given release. As an example, you may have a developer environment that's customized heavily. This environment shouldn't block a release to the test or pre-production environments if deployments often fail there due to components being out-of-standard.

### Conceptualize the workflow

After you have all your pieces, imagine or draw the workflow your deployments will go through as if it's a factory floor. There's a large machine at one end, pumping out artifacts. This is your CI pipeline. Track the path it takes to get to the end environments. Where do the conveyor belts diverge? How do you decide where an artifact goes? Who decides when it progresses? Who is responsible for approving a release at what times?

This process can help you understand your CI/CD pipeline as a whole and lets you see where there are bottlenecks or points of failure. Too much complexity to "make things faster" often slows you down. Try to prevent yourself from prematurely optimizing the workflow. There are always changes as you build the workflow out, and the less you lock in before you're sure about what you need, the better.


## Conclusion 

Kubernetes is complex, and choosing the right tooling across your pipeline is important. Often one tool won’t suit your needs across the CI/CD process. Using multiple tools across your pipeline makes getting the desired results easier.

This blog post was an excerpt from our new guide, Kubernetes delivery unlocked, which helps you understand:

- How to use Kubernetes' architecture to achieve seamless, zero-downtime deployments
- How to define and build your deployment process using CD principles
- Common challenges faced by developers using Kubernetes and how to overcome them

[Download the free guide, Kubernetes delivery unlocked](https://octopus.com/whitepapers/kubernetes-delivery-unlocked).

Happy deployments! 