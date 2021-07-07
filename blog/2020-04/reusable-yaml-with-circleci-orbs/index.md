---
title: Reusable YAML with CircleCI orbs
description: An overview of using and creating CircleCI Orbs.
author: ryan.rousseau@octopus.com
visibility: public
published: 2020-04-29
metaImage: octopus-circle-ci-orb.png
bannerImage: octopus-circle-ci-orb.png
bannerImageAlt: Reusable YAML with CircleCI orbs
tags:
 - DevOps
 - CircleCI
---

![Reusable YAML with CircleCI orbs](octopus-circle-ci-orb.png)

A growing trend among continuous integration and continuous delivery platforms is providing the ability to define pipelines as code, usually with YAML. One of the leaders in this area is CircleCI. In this post, we will look at a CircleCI configuration, including how to use and author CircleCI `orbs`. An orb is a reusable chunk of YAML that can be used across your CircleCI pipelines. Instead of copying and pasting the same code across multiple files, you can reference the functionality from the orb and keep your pipeline DRY.

## What is CircleCI

CircleCI is a continuous integration platform that uses YAML based configurations to execute jobs on Docker containers or virtual machines. These jobs can be woven together into one or more workflows that will execute when you commit your code to source control.

Below is an example CircleCI job named `build`:

```yaml
jobs:
  build:
    docker:
      - image: mcr.microsoft.com/dotnet/core/sdk:2.1.607-stretch
    environment:
      PACKAGE_VERSION: 1.3.<< pipeline.number >>
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
            dotnet cake build.cake --target="Publish" --packageVersion="$PACKAGE_VERSION"
      - persist_to_workspace:
          root: publish
          paths:
            - "*"
```

Let's break this down piece by piece.

First, we define which docker image to execute our job on:

```yaml
    docker:
      - image: mcr.microsoft.com/dotnet/core/sdk:2.1.607-stretch
```

We define an environment variable for our package version that includes the CircleCI pipeline number:

```yaml
    environment:
      PACKAGE_VERSION: 1.3.<< pipeline.number >>
```

We define our steps. The first step checks out our source code:

```yaml
    steps:
      - checkout
```

The second step installs Cake on the container executing the job:

```yaml
      - run:
          name: Install Cake
          command: |
            export PATH="$PATH:$HOME/.dotnet/tools"
            dotnet tool install -g Cake.Tool --version 0.35.0
```

The third step invokes cake to build our project:

```yaml
      - run:
          name: Build
          command: |
            export PATH="$PATH:$HOME/.dotnet/tools"
            dotnet cake build.cake --target="Publish" --packageVersion="$PACKAGE_VERSION"
```

The last step persists our published applications in the `publish` folder to a CircleCI workspace. Workspaces are used to share assets across multiple jobs:

```yaml
      - persist_to_workspace:
          root: publish
          paths:
            - "*"
```

Let's take a look at a workflow that uses this job. When defining the workflow's jobs, we can set a job to depend on one or more other jobs. We've created a workflow called `build` that will run the jobs `build`, `package`, `push`, `create-release`, and `deploy-release` in that order:

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

## Reusable YAML and CircleCI orbs

Let's say that we have multiple projects that use Cake to build, and the steps are pretty much the same. Wouldn't it be great for each project to use a standard process without defining the same YAML in each configuration?

That's where orbs come in! Orbs define reusable executors (the environment your job will run in), commands that can be used in jobs, and jobs that can be used in workflows.

### Using an orb

To use an orb, you import it with an `orbs` section in your configuration. In the section below, we've added the Slack orb by giving it the name `slack` and mapping it to the orb `circleci/slack@3.4.2`. `circleci` is the namespace that contains the orb. `slack` is the name of the orb. `3.4.2` is the version of the orb we want to use.

In our `build` job, we've added a new step `- slack\notify`. This step is the `notify` command from that orb. You can view the [source of the command](https://circleci.com/orbs/registry/orb/circleci/slack#commands-notify). That's quite a bit of YAML that we didn't need to include in our configuration.

```yaml
version: 2.1

orbs:
  slack: circleci/slack@3.4.2

jobs:
  build:
    docker:
      - image: mcr.microsoft.com/dotnet/core/sdk:2.1.607-stretch
    environment:
      PACKAGE_VERSION: 1.3.<< pipeline.number >>
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
            dotnet cake build.cake --target="Publish" --packageVersion="$PACKAGE_VERSION"
      - persist_to_workspace:
          root: publish
          paths:
            - OctopusSamples.OctoPetShop.Database
            - OctopusSamples.OctoPetShop.Infrastructure
            - OctopusSamples.OctoPetShop.ProductService
            - OctopusSamples.OctoPetShop.ShoppingCartService
            - OctopusSamples.OctoPetShop.Web
      - slack/notify:
          message: Build ${PACKAGE_VERSION} succeeded
```

So let's create an orb for our Cake steps.

### Creating an inline orb

In the example above, we imported an orb into our configuration. We can also define an orb directly in our configuration:

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
            default: "$PACKAGE_VERSION"
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

In the orb above, we've created an executor that defines the Docker image to use. We also defined a `build` job that accepts the Cake version, package version, publish path, and Cake target as parameters. It then uses those parameters in steps similar to those we had before.

With the orb defined, we can update our workflow to the following:

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

### Publishing an orb

To use this orb in multiple projects, we need to publish it. To do that, we need to install and configure the [CircleCI CLI](https://circleci.com/docs/2.0/orb-author-cli/#install-the-cli-for-the-first-time).

After that's done, we create a namespace for our organization:

```bash
circleci namespace create octopus-samples github OctopusSamples # replace with your values
```

Then we create the orb in our namespace:

```bash
circleci orb create octopus-samples/cake # replace with your values
```

Now to get that inline orb published to CircleCI. We need to move our orb definition from our configuration into its own YAML file. We'll name it orb.yml.

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
        default: "$PACKAGE_VERSION"
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

And then we can publish it with:

```bash
circleci orb publish orb.yml octopus-samples/cake@0.0.1
```

And presto! We have a [published orb](https://circleci.com/orbs/registry/orb/octopus-samples/cake) with our Cake build job. Now we can update our project's configuration to replace the inline orb with the published one:

```yaml
orbs:
  cake: octopus-samples/cake@0.0.1
```

## Experimental Octopus CLI orb

Speaking of published orbs, if you're using CircleCI and Octopus together, take a peek at our [experimental Octopus CLI orb](https://circleci.com/orbs/registry/orb/octopusdeploylabs/octopus-cli). With this orb, you can use a subset of commands from the Octopus CLI to manage your packages, releases, and deployments:

```yaml
orbs:
  octo: octopusdeploylabs/octopus-cli@0.0.2

jobs:
  package:
    docker:
      - image: ubuntu:18.04
    environment:
      PACKAGE_VERSION: 1.3.<< pipeline.number >>
    steps:
      - octo/install-tools
      - attach_workspace:
          at: publish
      - octo/pack:
          id: "OctopusSamples.OctoPetShop.Database"
          version: "$PACKAGE_VERSION"
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
      PACKAGE_VERSION: 1.3.<< pipeline.number >>
    steps:
      - octo/install-tools
      - attach_workspace:
          at: package
      - octo/push:
          package: "package/OctopusSamples.OctoPetShop.Database.${PACKAGE_VERSION}.zip"
          server: "$OCTOPUS_SERVER"
          api_key: "$OCTOPUS_API_KEY"
          debug: true
      - octo/build-information:
          package_id: "OctopusSamples.OctoPetShop.Database"
          version: "$PACKAGE_VERSION"
          server: "$OCTOPUS_SERVER"
          api_key: "$OCTOPUS_API_KEY"
          debug: true
  create-release:
    docker:
      - image: ubuntu:18.04
    environment:
      PACKAGE_VERSION: 1.3.<< pipeline.number >>
    steps:
      - octo/install-tools
      - octo/create-release:
          project: "Octo Pet Shop"
          server: "$OCTOPUS_SERVER"
          api_key: "$OCTOPUS_API_KEY"
          release_number: $PACKAGE_VERSION
```

As a bonus, we can continue using an inline orb to reduce our duplication even more. Let's make a wrapper around `octo-exp\pack`:

```yaml
orbs:
  cake: octopus-samples/cake@0.0.1
  octo: octopusdeploylabs/octopus-cli@0.0.2
  octopetshop:
    orbs:
      octo: octopusdeploylabs/octopus-cli@0.0.2
    commands:
      pack:
        parameters:
          id:
            type: string
        steps:
          - octo/pack:
              id: "<< parameters.id >>"
              version: $PACKAGE_VERSION
              base_path: "publish/<< parameters.id >>"
              out_folder: "package"

jobs:
  package:
    executor:
      - name: octo/default
    environment:
      PACKAGE_VERSION: 1.3.<< pipeline.number >>
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

CircleCI Orbs are a way to create reusable commands or jobs for your YAML based pipelines. You can publish orbs to CircleCI to share across projects or with other organizations. You can also use inline orbs to aid in development or to cut down the noise in your own configurations.

Check out [CircleCI Orbs](https://circleci.com/orbs/) for more information and the [experimental Octopus CLI orb](https://circleci.com/orbs/registry/orb/octopusdeploylabs/octopus-cli) for details on how you can use CircleCI and Octopus together.
