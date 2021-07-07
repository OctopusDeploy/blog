---
title: Publishing a package to Octopus with GitHub Actions
description: A post showing how to create a GitHub Action that pushes a package to Octopus.
author: ryan.rousseau@octopus.com
visibility: public
published: 2020-04-27
metaImage: github-actions-publish.png
bannerImage: github-actions-publish.png
bannerImageAlt: Publishing a package to Octopus with GitHub Actions
tags:
 - DevOps
---

![Publishing a package to Octopus with GitHub Actions](github-actions-publish.png)

:::hint
As part of the Octopus 2021 Q2 release, we introduced native support for GitHub Actions so you can integrate your GitHub builds and other automated processes with your Octopus workflows. Learn how to get started in our post [Announcing GitHub Actions for Octopus Deploy](https://octopus.com/blog/github-actions-for-octopus-deploy).
:::

I recently set aside some time to write my first GitHub Action. I used my personal blog as my test case. It is a static site that is built with Wyam and hosted in Firebase.

[TL;DR](#full-script): skip down to the bottom for the full script.

## What are GitHub Actions?

GitHub Actions are workflows that can react to events raised in your repository. Pushing to a branch, opening a pull request, and opening an issue are examples of events that can trigger your workflow.

In my case, my workflow reacts to pushing to the master branch.

The workflow contains one or more jobs that run on Docker containers. Jobs contain one or more steps that carry out the tasks to perform the required action.

## My workflow

The high-level set of steps that I need to perform are:

* Check out the code.
* Generate the site.
* Package the site.
* Push that package to my Octopus instance.

Some of these steps require other steps to run first. I'll need steps to install .NET Core, Wyam, and the Octopus CLI. I also need a step to calculate a version for my site package.

:::hint
The sample workflow in this post used a now deprecrated [set-env command](https://github.blog/changelog/2020-10-01-github-actions-deprecating-set-env-and-add-path-commands/). The post has been updated to use [environment files](https://docs.github.com/en/actions/reference/workflow-commands-for-github-actions#environment-files) instead.
:::

## Workflow creation

Creating a new workflow was pretty intuitive. I went to the **Actions** tab of my repository and clicked the **New workflow** button.

There are some starter workflows that you can choose. I skipped those and went with the **Set up a workflow yourself** option.

GitHub launched me into an editor for a new YAML file in the `.github/workflows/` folder of my repository.

Here are the changes I made to that file.

### Name and trigger

I kept the default name `CI` and the trigger for pushes to the `master` branch:

```yml
name: CI

on:
  push:
    branches: [ master ]
```

### Generate job

I renamed the default job from `build` to `generate` and accepted the image it runs on: `ubuntu-latest`:

```yml
jobs:
  generate:
    runs-on: ubuntu-latest
```

### Checkout step

Below is the most minimal step you can add to a job. In this one line, I defined a step that checks out my repository to the Ubuntu container hosting the job.

Reusing steps is a key feature of Actions and many other YAML based pipelines. I use version 2 of the `checkout` step provided by the GitHub Actions team. You can also author actions and use third party actions in your workflows:

```yml
steps:
    - uses: actions/checkout@v2
```

### Workflow commands and environment variables

Actions provide [workflow commands](https://help.github.com/en/actions/reference/workflow-commands-for-github-actions) that you can use in your steps with a special string format passed to `echo`.

This step is called **Set version** and creates a package version based on the current date and the current run number of the workflow. The package version is saved as `$PACKAGE_VERSION` using the `set-env` command. `$PACKAGE_VERSION` is now available in the steps that create and publish the package:

```yml
    - name: Set version
      run: echo "PACKAGE_VERSION=$(date +'%Y.%m.%d').$GITHUB_RUN_NUMBER" >> $GITHUB_ENV
```

### Generate the site

To generate my static site, I need to install .NET Core, the Wyam tool, and call Wyam against my source.

I use another built-in action `actions/setup-dotnet@v1` to install .NET Core on the container. This step accepts a parameter for the .NET version to install:

```yml
    - name: Setup .NET Core
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: 2.1.802
```

With .NET installed, I can call the CLI to install the Wyam tool onto the container:

```yml
    - name: Install Wyam
      run: dotnet tool install --tool-path . wyam.tool
```

Finally, the `Generate site` step calls Wyam to generate the site:

```yml
    - name: Generate site
      run: ./wyam build -r blog -t Stellar src
```

### Package and publish the site

I need the Octopus CLI to package and publish the site. I hopped over to the [Octopus CLI download page](https://octopus.com/downloads/octopuscli#linux) and copied the script for installing it on Ubuntu:

```yml
    - name: Install Octopus CLI
      run: |
        sudo apt update && sudo apt install --no-install-recommends gnupg curl ca-certificates apt-transport-https && \
        curl -sSfL https://apt.octopus.com/public.key | sudo apt-key add - && \
        sudo sh -c "echo deb https://apt.octopus.com/ stable main > /etc/apt/sources.list.d/octopus.com.list" && \
        sudo apt update && sudo apt install octopuscli
```

I copy the necessary files into a directory representing the package. Then I call the `octo pack` command to create a Zip package with the version set to `$PACKAGE_VERSION`:

```yml
    - name: Package blog.rousseau.dev
      run: |
        mkdir -p ./packages/blog.rousseau.dev/src/
        cp .firebaserc firebase.json ./packages/blog.rousseau.dev/
        cp -avr ./src/output ./packages/blog.rousseau.dev/src/
        octo pack --id="blog.rousseau.dev" --format="Zip" --version="${{ env.PACKAGE_VERSION }}" --basePath="./packages/blog.rousseau.dev" --outFolder="./packages"
```

A snippet of the output from this step looks like:

```
Setting Zip compression level to Optimal
Packing blog.rousseau.dev version "2020.04.08.21"...
Saving "blog.rousseau.dev.2020.04.08.21.zip" to "./packages"...
Adding files from "./packages/blog.rousseau.dev" matching pattern "**"
```

With the package created, I call the `octo push` command to upload the package to my Octopus instance. My server URL and API key are configured as secrets in my repository to keep them safe and secure:

```yml
    - name: Push blog.rousseau.dev to Octopus
      run: octo push --package="./packages/blog.rousseau.dev.${{ env.PACKAGE_VERSION }}.zip" --server="${{ secrets.OCTOPUS_SERVER_URL }}" --apiKey="${{ secrets.OCTOPUS_API_KEY }}"
```

On the topic of API keys, I _highly_ recommend setting up a [service account](https://octopus.com/docs/administration/managing-users-and-teams/service-accounts) when integrating another application with Octopus. There's a built-in `Build server` role that you can use for CI service accounts.

The results below show the request is authenticating as `GitHub (a service account)` instead of my user account.

The package that was created in the package step was successfully pushed to Octopus. Now I can deploy my site to Firebase (but that is a post for another time):

```
Detected automation environment: "NoneOrUnknown"
Space name unspecified, process will run in the default space context
Handshaking with Octopus Server: ***
Handshake successful. Octopus version: 2020.1.6; API version: 3.0.0
Authenticated as: GitHub (a service account)
Pushing package: /home/runner/work/blog.rousseau.dev/blog.rousseau.dev/packages/blog.rousseau.dev.2020.04.08.21.zip...
Requesting signature for delta compression from the server for upload of a package with id 'blog.rousseau.dev' and version '2020.04.08.21'
Calculating delta
The delta file (60,053 bytes) is 0.42 % the size of the original file (14,338,268 bytes), uploading...
Delta transfer completed
Push successful
```

## Full script

```yml
name: CI

on:
  push:
    branches: [ master ]

jobs:
  generate:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2

    - name: Set version
      id: set-version
      run: echo "PACKAGE_VERSION=$(date +'%Y.%m.%d').$GITHUB_RUN_NUMBER" >> $GITHUB_ENV

    - name: Setup .NET Core
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: 2.1.802

    - name: Install Wyam
      run: dotnet tool install --tool-path . wyam.tool

    - name: Generate site
      run: ./wyam build -r blog -t Stellar src

    - name: Install Octopus CLI
      run: |
        sudo apt update && sudo apt install --no-install-recommends gnupg curl ca-certificates apt-transport-https && \
        curl -sSfL https://apt.octopus.com/public.key | sudo apt-key add - && \
        sudo sh -c "echo deb https://apt.octopus.com/ stable main > /etc/apt/sources.list.d/octopus.com.list" && \
        sudo apt update && sudo apt install octopuscli

    - name: Package blog.rousseau.dev
      run: |
        mkdir -p ./packages/blog.rousseau.dev/src/
        cp .firebaserc firebase.json ./packages/blog.rousseau.dev/
        cp -avr ./src/output ./packages/blog.rousseau.dev/src/
        octo pack --id="blog.rousseau.dev" --format="Zip" --version="${{ env.PACKAGE_VERSION }}" --basePath="./packages/blog.rousseau.dev" --outFolder="./packages"

    - name: Push blog.rousseau.dev to Octopus
      run: octo push --package="./packages/blog.rousseau.dev.${{ env.PACKAGE_VERSION }}.zip" --server="${{ secrets.OCTOPUS_SERVER_URL }}" --apiKey="${{ secrets.OCTOPUS_API_KEY }}"
```

## Conclusion

GitHub Actions are an excellent way to add continuous integration and delivery directly to your projects hosted on GitHub. When combined with the Octopus CLI, you've got a powerful onramp to repeatable, reliable deployments.
