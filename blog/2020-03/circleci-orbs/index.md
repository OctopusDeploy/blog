---
title: Reusable YAML with CircleCI Orbs
description: An overview of using and creating CircleCI Orbs
author: ryan.rousseau@octopus.com
visibility: private
published: 2023-03-30
metaImage:
bannerImage:
tags:
 - DevOps
---

YAML based build pipelines are increasing in popularity. They enable committing your build configuration into source control alongside your application code. This has many benefits such as using the same review process for your pipelines as you do for code, being able to view change history and revert to a previous version, etc.

When working with YAML pipelines, you might need to use the same snippet of YAML multiple times in one pipeline or to share a snippet across multiple pipelines or projects. CircleCI solves this problem with an offering called orbs.

## What is CircleCI

CircleCI is a continuous integration platform that uses YAML based configurations to execute jobs on Docker containers or virtual machines. These jobs can be woven together into one or more workflows that will execute when you commit your code to source control.

Here is an example CircleCI job.

```yaml
jobs:
  build:
    docker:
      - image: mcr.microsoft.com/dotnet/core/sdk:2.1.607-stretch
    environment:
      PACKAGEVERSION: 1.3.<< pipeline.number >>
    steps:
      - checkout
      - run:
          name: Install Cake
          command: |
            export PATH="$PATH:$HOME/.dotnet/tools"
            dotnet tool install -g Cake.Tool --version 0.35.0
      - run:
          name: Build
          command: |
            export PATH="$PATH:$HOME/.dotnet/tools"
            dotnet cake build.cake --target="Publish" --packageVersion="$PACKAGEVERSION"
      - persist_to_workspace:
          root: publish
          paths:
            - OctopusSamples.OctoPetShop.Database
            - OctopusSamples.OctoPetShop.Infrastructure
            - OctopusSamples.OctoPetShop.ProductService
            - OctopusSamples.OctoPetShop.ShoppingCartService
            - OctopusSamples.OctoPetShop.Web
```

We define what docker image we want to execute our job on - `mcr.microsoft.com/dotnet/core/sdk:2.1.607-stretch`. Then we define an environment variable representing the package version we want to use during this job. It is based on the number of the pipeline that is currently running. Finally, we define our steps.

This job will checkout our source code, install Cake on the container, use Cake to build our application, and then persist the published packages to a CircleCI workspace where they can be used by subsequent jobs in the workflow.

And here is an example workflow that uses this job.

```yaml
workflows:
  version: 2
  build:
    jobs:
      - build
      - package:
          requires:
            - build
      - push:
          requires:
            - package
      - create-release:
          requires:
            - push
      - deploy-release:
          requires:
            - create-release
```

When defining the workflow's jobs, we can set one job to require one or more other jobs to complete. Doing this, we've created a workflow named `build` that will run the jobs `build`, `package`, `push`, `create-release`, and `deploy-release` in that order.

## Reusable YAML and CircleCI Orbs

Let's say that we have multiple projects that use Cake to build and the steps are pretty much the same. Wouldn't it be great for each project to be able to use a common process without defining the same YAML in each configuration?

That's where orbs come in! Orbs define reusable executors (the environment your job will run in), commands that can be used in jobs, and jobs that can be used in workflows.

### Using an Orb

To use an orb, you'll import it with an `orbs` section in your configuration. In the section below, we've added the Slack orb by giving it the name `slack` and mapping it to the orb `circleci/slack@3.4.2`. `circleci` is the namespace that contains the orb. `slack` is the name of the orb. `3.4.2` is the version of the orb that we want to use.

In our `build` job, we've added a new step `- slack\notify`. This is using the `notify` command from that orb. You can view the source of the command [here](https://circleci.com/orbs/registry/orb/circleci/slack#commands-notify). That's quite a bit of YAML that we didn't need to include in our configuration.

```yaml
version: 2.1

orbs:
  slack: circleci/slack@3.4.2

jobs:
  build:
    docker:
      - image: mcr.microsoft.com/dotnet/core/sdk:2.1.607-stretch
    environment:
      PACKAGEVERSION: 1.3.<< pipeline.number >>
    steps:
      - checkout
      - run:
          name: Install Cake
          command: |
            export PATH="$PATH:$HOME/.dotnet/tools"
            dotnet tool install -g Cake.Tool --version 0.35.0
      - run:
          name: Build
          command: |
            export PATH="$PATH:$HOME/.dotnet/tools"
            dotnet cake build.cake --target="Publish" --packageVersion="$PACKAGEVERSION"
      - persist_to_workspace:
          root: publish
          paths:
            - OctopusSamples.OctoPetShop.Database
            - OctopusSamples.OctoPetShop.Infrastructure
            - OctopusSamples.OctoPetShop.ProductService
            - OctopusSamples.OctoPetShop.ShoppingCartService
            - OctopusSamples.OctoPetShop.Web
      - slack/notify:
          message: Build ${PACKAGEVERSION} succeeded
```

So let's create an orb for our Cake steps.

### Creating an Inline Orb

In the example above, we imported an orb into our configuration. We can also define an orb directly in our configuration.

```yaml
orbs:
  cake:
    executors:
      cake-executor:
        docker:
          - image: mcr.microsoft.com/dotnet/core/sdk:2.1.607-stretch
    jobs:
      build:
        executor: cake-executor
        parameters:
          cake_version:
            default: "0.35.0"
            type: string
          package_version:
            default: "$PACKAGEVERSION"
            type: string
          publish_path:
            default: "publish"
            type: string
          target:
            default: "Publish"
            type: string
        steps:
          - checkout
          - run:
              name: Install Cake
              command: |
                export PATH="$PATH:$HOME/.dotnet/tools"
                dotnet tool install -g Cake.Tool --version << parameters.cake_version >>
          - run:
              name: Build
              command: |
                export PATH="$PATH:$HOME/.dotnet/tools"
                dotnet cake build.cake --target="<< parameters.target >>" --packageVersion="<< parameters.package_version >>"
          - persist_to_workspace:
              root: << parameters.publish_path >>
              paths:
                - "*"
```

In the orb above, we've created an executor that defines the Docker image to use. We also defined a `build` job that accepts the Cake version, package version, publish path, and Cake target as parameters. It then uses those parameters in steps similar to what we had before.

With the orb defined, we can update our workflow to the following.

```yaml
workflows:
  version: 2
  build:
    jobs:
      - cake/build:
          package_version: 1.3.<< pipeline.number >>
      - package:
          requires:
            - cake/build
      - push:
          requires:
            - package
      - create-release:
          requires:
            - push
      - deploy-release:
          requires:
            - create-release
```

Now our workflow will use the `build` job from the Cake orb we defined in our configuration.

## Publishing an Orb

For our team (and others) to use this orb, we'll need to publish it. To do that, we'll need to install and configure the [CircleCI CLI](https://circleci.com/docs/2.0/orb-author-cli/#install-the-cli-for-the-first-time).

Once that's done, we'll create a namespace for our organization.

```bash
circleci namespace create octopus-samples github OctopusSamples # replace with your values
```

Then we create the orb in our namespace.

```bash
circleci orb create octopus-samples/cake # replace with your values
```

Now to get that inline orb published to CircleCI. We need to move our orb definition from our configuration into it's own yml file. We'll name it orb.yml.

```yaml
version: 2.1

description: |
  Orb for using Cake with OctopusSamples.

executors:
  cake-executor:
    docker:
      - image: mcr.microsoft.com/dotnet/core/sdk:2.1.607-stretch
jobs:
  build:
    executor: cake-executor
    parameters:
      cake_version:
        default: "0.35.0"
        type: string
      package_version:
        default: "$PACKAGEVERSION"
        type: string
      publish_path:
        default: "publish"
        type: string
      target:
        default: "Publish"
        type: string
    steps:
      - checkout
      - run:
          name: Install Cake
          command: |
            export PATH="$PATH:$HOME/.dotnet/tools"
            dotnet tool install -g Cake.Tool --version << parameters.cake_version >>
      - run:
          name: Build
          command: |
            export PATH="$PATH:$HOME/.dotnet/tools"
            dotnet cake build.cake --target="<< parameters.target >>" --packageVersion="<< parameters.package_version >>"
      - persist_to_workspace:
          root: << parameters.publish_path >>
          paths:
            - "*"
```

And then we can publish it with

```bash
circleci orb publish orb.yml octopus-samples/cake@0.0.1
```

And presto! We have a [published orb](https://circleci.com/orbs/registry/orb/octopus-samples/cake) with our Cake build job. Now we can update our project's configuration to replace the inline orb with the published one.

```yaml
orbs:
  cake: octopus-samples/cake@0.0.1
```

## Experimental Octo CLI Orb

And speaking of published orbs, if you're using CircleCI and Octopus together, take a peek at our [experimental Octopus CLI orb](https://circleci.com/orbs/registry/orb/octopus-samples/octo-exp). With this orb, you can use a subset of commands from the Octo CLI to manage your packages, releases, and deployments.

```yaml
orbs:
  octo: octopus-samples/octo-exp@0.0.2

jobs:
  package:
    docker:
      - image: ubuntu:18.04
    environment:
      PACKAGEVERSION: 1.3.<< pipeline.number >>
    steps:
      - octo/install-tools
      - attach_workspace:
          at: publish
      - octo/pack:
          id: "OctopusSamples.OctoPetShop.Database"
          version: "$PACKAGEVERSION"
          base_path: "publish/OctopusSamples.OctoPetShop.Database"
          out_folder: "package"
      - persist_to_workspace:
          root: package
          paths:
            - OctopusSamples*
  push:
    docker:
      - image: ubuntu:18.04
    environment:
      PACKAGEVERSION: 1.3.<< pipeline.number >>
    steps:
      - octo/install-tools
      - attach_workspace:
          at: package
      - octo/push:
          package: "package/OctopusSamples.OctoPetShop.Database.${PACKAGEVERSION}.zip"
          server: "$OCTOPUS_SERVER"
          api_key: "$OCTOPUS_API_KEY"
          debug: true
      - octo/build-information:
          package_id: "OctopusSamples.OctoPetShop.Database"
          version: "$PACKAGEVERSION"
          server: "$OCTOPUS_SERVER"
          api_key: "$OCTOPUS_API_KEY"
          debug: true
  create-release:
    docker:
      - image: ubuntu:18.04
    environment:
      PACKAGEVERSION: 1.3.<< pipeline.number >>
    steps:
      - octo/install-tools
      - octo/create-release:
          project: "Octo Pet Shop"
          server: "$OCTOPUS_SERVER"
          api_key: "$OCTOPUS_API_KEY"
          release_number: $PACKAGEVERSION
```

As a bonus, we can continue using an inline orb to reduce our duplication even more. Let's make a wrapper around `octo-exp\pack`.

```yaml
orbs:
  cake: octopus-samples/cake@0.0.1
  octo: octopus-samples/octo-exp@0.0.2
  octopetshop:
    orbs:
      octo: octopus-samples/octo-exp@0.0.2
    commands:
      pack:
        parameters:
          id:
            type: string
        steps:
          - octo/pack:
              id: "<< parameters.id >>"
              version: $PACKAGEVERSION
              base_path: "publish/<< parameters.id >>"
              out_folder: "package"

jobs:
  package:
    docker:
      - image: ubuntu:18.04
    environment:
      PACKAGEVERSION: 1.3.<< pipeline.number >>
    steps:
      - octo/install-tools
      - attach_workspace:
          at: publish
      - octopetshop/pack:
          id: "OctopusSamples.OctoPetShop.Database"
      - octopetshop/pack:
          id: "OctopusSamples.OctoPetShop.Infrastructure"
      - octopetshop/pack:
          id: "OctopusSamples.OctoPetShop.ProductService"
      - octopetshop/pack:
          id: "OctopusSamples.OctoPetShop.ShoppingCartService"
      - octopetshop/pack:
          id: "OctopusSamples.OctoPetShop.Web"
      - persist_to_workspace:
          root: package
          paths:
            - OctopusSamples*
```

Since our calls to octo/pack will follow the same format with different values for `id`, we can use `octopetshop\pack` to abstract that logic away.

## Conclusion

CircleCI Orbs are a way to create reusuable commands or jobs for your YAML based pipelines. You can publish orbs to CircleCI to share across projects or with other organizations. You can also use inline orbs to aid in development or to cut down the noise in your own configurations.

Check out [CircleCI Orbs] for more information and the [experimental Octo CLI orb](https://circleci.com/orbs/registry/orb/octopus-samples/octo-exp) for details on how you can use CircleCI and Octopus together.
