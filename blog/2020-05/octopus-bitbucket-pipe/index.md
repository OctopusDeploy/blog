---
title: "Octopus Pipe for Bitbucket: octopus-cli-run"
description: Learn how to integrate Octopus and BitBucket with our new experimental BitBucket Pipe called octopus-cli-run.
author: mark.harrison@octopus.com
visibility: public
published: 2020-05-27
metaImage: bitbucket-cd.png
bannerImage: bitbucket-cd.png
bannerImageAlt: Bitbucket Pipelines
tags:
 - DevOps
---

![Bitbucket Pipelines](bitbucket-cd.png)

In a previous post, I wrote [how to create a Bitbucket Pipe and integrate it with Octopus Deploy](blog/2020-04/bitbucket-pipes-and-octopus-deploy/index.md). If you’re starting out with Pipes for the first time, it’s worth a read.

In this post, I’ll give you an overview of the new experimental Bitbucket Pipe for Octopus - [octopus-cli-run](https://bitbucket.org/octopusdeploy/octopus-cli-run/).
If you’re interested in trying the experimental pipe, you can use it to run commands from the [Octopus CLI](https://octopus.com/docs/octopus-rest-api/octopus-cli/), allowing you to further integrate your Atlassian [Bitbucket Pipeline](https://bitbucket.org/product/features/pipelines) with Octopus to manage your packages, releases, and deployments.

<h2>In this post</h2>

!toc

## Pipe YAML Definition

The base definition of the Pipe includes the reference to its repository hosted on [Bitbucket](https://bitbucket.org/octopusdeploy/octopus-cli-run/). It has also been published as [octopipes/octopus-cli-run](https://hub.docker.com/r/octopipes/octopus-cli-run/) on Docker Hub.

It has one required `CLI_COMMAND` variable. This is the CLI command to run.

To use the Pipe in your `bitbucket-pipelines.yml` file, add the following YAML snippet to the script section:

```yaml
- pipe: octopusdeploy/octopus-cli-run:0.13.0
  variables:
    CLI_COMMAND: "<string>"
    # EXTRA_ARGS: ['<string>','<string>' ..] # Optional
    # DEBUG: "<boolean>" # Optional
```

The Pipe also provides an *optional* array variable called `EXTRA_ARGS` that you can use to include any additional command line arguments for the specified command.

### Pipe variable definitions

Variables in Bitbucket Pipelines and Pipes are configured as [Environment variables](https://confluence.atlassian.com/bitbucket/variables-in-pipelines-794502608.html). As the `octopus-cli-run` Pipe contains a number of commands, the specific variables that are required depend on which command you are using. See the [README](https://bitbucket.org/octopusdeploy/octopus-cli-run/src/master/README.md#markdown-header-variables) for further details of the variables that are required for each command.

## Supported commands

The `octopus-cli-run` Pipe was written with the most commonly used CLI commands in mind, and it’s actually built on top of the [Octopus CLI Docker image](https://hub.docker.com/r/octopusdeploy/octo/). This includes the ability to:

 - Package your files or build artifacts using [pack](https://octopus.com/docs/octopus-rest-api/octopus-cli/pack).
 - Send packages to the Octopus built-in repository using [push](https://octopus.com/docs/octopus-rest-api/octopus-cli/pack).
 - Push build information to Octopus using [build-information](https://octopus.com/docs/octopus-rest-api/octopus-cli/build-information).
 - Automate creation of releases using [create-release](https://octopus.com/docs/octopus-rest-api/octopus-cli/create-release).
 - Deploy releases that have already been created using [deploy-release](https://octopus.com/docs/octopus-rest-api/octopus-cli/create-release).

Next, we’ll explore what the Pipeline steps look like for each of the commands using the **PetClinic** sample application available on [Bitbucket](https://bitbucket.org/octopussamples/petclinic/). To keep it simple, the steps have been reduced to the minimum definition that is needed.

### Pack

The `pack` command allows you to create [packages](https://octopus.com/docs/packaging-applications) (either as zip or nupkg) from files on disk, without the need for a `.nuspec` or `.csproj` file.

To create a package, define a step like this:

```yaml
- step:
    name: octo pack mysql-flyway
    script:
      - pipe: octopusdeploy/octopus-cli-run:0.13.0
        variables:
          CLI_COMMAND: 'pack'
          ID: 'petclinic.mysql.flyway'
          FORMAT: 'Zip'
          VERSION: '1.0.0.0'
          SOURCE_PATH: 'flyway'
          OUTPUT_PATH: 'flyway'
    artifacts:
      - "flyway/*.zip"
```
This packages the `flyway` folder and creates a zip file named `petclinic.mysql.flyway.1.0.0.0.zip` in the same folder.

### Push

The `push` command enables you to push packages (.zip, .nupkg, .war, etc) to the Octopus [built-in repository](https://octopus.com/docs/packaging-applications/package-repositories/built-in-repository)

It also supports pushing multiple packages at the same time. To perform a multi-package `push`, define a step like this:

```yaml
- step:
    name: octo push
    script:
      - pipe: octopusdeploy/octopus-cli-run:0.13.0
        variables:
          CLI_COMMAND: 'push'
          OCTOPUS_SERVER: $OCTOPUS_SERVER
          OCTOPUS_APIKEY: $OCTOPUS_API_KEY
          OCTOPUS_SPACE: $OCTOPUS_SPACE
          PACKAGES: [ "./flyway/petclinic.mysql.flyway.1.0.0.0.zip", "target/petclinic.web.1.0.0.0.war" ]
```

This pushes both the `petclinic.mysql.flyway.1.0.0.0.zip` and the `petclinic.web.1.0.0.0.war` packages to Octopus.

### Build information

The `build-information` command helps you to pass information about your build (number, URL, commits) to Octopus. This information can be viewed within Octopus, and can also be used in both [release notes](https://octopus.com/docs/managing-releases/release-notes) and [deployment notes](https://octopus.com/docs/managing-releases/deployment-notes).

If you have already created a build-information file, you can supply this to the command using the `FILE` variable. If the variable isn’t provided, the Pipe will generate its own build information file and send it to Octopus.

To push an auto-generated build info file, define a step like this:

```yaml
- step:
    name: octo build-information
    script:
      - pipe: octopusdeploy/octopus-cli-run:0.13.0
        variables:
          CLI_COMMAND: 'build-information'
          OCTOPUS_SERVER: $OCTOPUS_SERVER
          OCTOPUS_APIKEY: $OCTOPUS_API_KEY
          OCTOPUS_SPACE: $OCTOPUS_SPACE
          VERSION: '1.0.0.0'
          PACKAGE_IDS: ['petclinic.web']
```

This creates build information, associates it with version `1.0.0.0` of the `petclinic.web` package, and pushes it to Octopus.

### Create release

The `create-release` command allows you to create a release in Octopus. You specify the project to create the release for using the `PROJECT` variable.

Optionally, you can also deploy the release to one or more environments. To achieve this, you should use the global `EXTRA_ARGS` array variable and provide the appropriate options. For example:

`EXTRA_ARGS: ['--deployTo', 'Development', '--guidedFailure', 'True']`

To create a release, and let Octopus choose the version to use, create a step like this:

```yaml
- step:
    name: octo create-release
    script:
      - pipe: octopusdeploy/octopus-cli-run:0.13.0
        variables:
          CLI_COMMAND: 'create-release'
          OCTOPUS_SERVER: $OCTOPUS_SERVER
          OCTOPUS_APIKEY: $OCTOPUS_API_KEY
          OCTOPUS_SPACE: $OCTOPUS_SPACE
          PROJECT: $OCTOPUS_PROJECT
```

### Deploy release

The `deploy-release` command lets you deploy releases that have already been created. You specify the project and the release number to deploy the release for using the `PROJECT` and `RELEASE_NUMBER` variables.

Choose the environment(s) to deploy to by specifying them in the `DEPLOY_TO` variable using either the Name or ID, like so:

`DEPLOY_TO: ['Environments-1', 'Development', 'Staging', 'Test']`

To deploy the `latest` release to `Development` for a project, create a step like this:

```yaml
- step:
    name: octo deploy-release
    script:
      - pipe: octopusdeploy/octopus-cli-run:0.13.0
        variables:
          CLI_COMMAND: 'deploy-release'
          OCTOPUS_SERVER: $OCTOPUS_SERVER
          OCTOPUS_APIKEY: $OCTOPUS_API_KEY
          OCTOPUS_SPACE: $OCTOPUS_SPACE
          PROJECT: $OCTOPUS_PROJECT
          RELEASE_NUMBER: 'latest'
          DEPLOY_TO: ['Development']
```

## Using the Pipe

Finally, let’s see the use of the Pipe in multiple steps to make up the complete Bitbucket Pipeline:

```yaml
image: maven:3.6.1

pipelines:
  branches:
    master:
      - step:
          name: build petclinic
          caches:
            - maven
          script:
            - mvn -B verify -DskipTests -Dproject.versionNumber=1.0.0.0 -DdatabaseUserName=$DatabaseUserName -DdatabaseUserPassword=$DatabaseUserPassword -DdatabaseServerName=$DatabaseServerName -DdatabaseName=$DatabaseName
          artifacts:
          - "target/*.war"
      - step:
          name: octo pack mysql-flyway
          script:
            - pipe: octopusdeploy/octopus-cli-run:0.13.0
              variables:
                CLI_COMMAND: 'pack'
                ID: 'petclinic.mysql.flyway'
                FORMAT: 'Zip'
                VERSION: '1.0.0.0'
                SOURCE_PATH: 'flyway'
                OUTPUT_PATH: './flyway'
          artifacts:
            - "flyway/*.zip"
      - step:
          name: octo push
          script:
            - pipe: octopusdeploy/octopus-cli-run:0.13.0
              variables:
                CLI_COMMAND: 'push'
                OCTOPUS_SERVER: $OCTOPUS_SERVER
                OCTOPUS_APIKEY: $OCTOPUS_API_KEY
                OCTOPUS_SPACE: $OCTOPUS_SPACE
                PACKAGES: [ "./flyway/petclinic.mysql.flyway.1.0.0.0.zip", "target/petclinic.web.1.0.0.0.war" ]
      - step:
          name: octo build-information
          script:
            - pipe: octopusdeploy/octopus-cli-run:0.13.0
              variables:
                CLI_COMMAND: 'build-information'
                OCTOPUS_SERVER: $OCTOPUS_SERVER
                OCTOPUS_APIKEY: $OCTOPUS_API_KEY
                OCTOPUS_SPACE: $OCTOPUS_SPACE
                VERSION: '1.0.0.0'
                PACKAGE_IDS: ['petclinic.web']
      - step:
          name: octo create-release
          script:
            - pipe: octopusdeploy/octopus-cli-run:0.13.0
              variables:
                CLI_COMMAND: 'create-release'
                OCTOPUS_SERVER: $OCTOPUS_SERVER
                OCTOPUS_APIKEY: $OCTOPUS_API_KEY
                OCTOPUS_SPACE: $OCTOPUS_SPACE
                PROJECT: $OCTOPUS_PROJECT
      - step:
          name: octo deploy-release
          script:
            - pipe: octopusdeploy/octopus-cli-run:0.13.0
              variables:
                CLI_COMMAND: 'deploy-release'
                OCTOPUS_SERVER: $OCTOPUS_SERVER
                OCTOPUS_APIKEY: $OCTOPUS_API_KEY
                OCTOPUS_SPACE: $OCTOPUS_SPACE
                PROJECT: $OCTOPUS_PROJECT
                RELEASE_NUMBER: 'latest'
                DEPLOY_TO: ['Development']
```
And that’s it!

You can view the complete PetClinic `bitbucket-pipelines.yml` file on [Bitbucket](https://bitbucket.org/octopussamples/petclinic/src/master/bitbucket-pipelines.yml).

:::success
**Sample Octopus project**
You can see the PetClinic Octopus project in our [samples](https://g.octopushq.com/TargetWildflySamplePetClinic) instance.
:::

## Conclusion

Using a Bitbucket Pipe really helps to simplify the configuration in your Bitbucket Pipeline, and as an author helps to promote re-use of your actions. Check out [Bitbucket Pipelines](https://bitbucket.org/product/features/pipelines/integrations) for more information and the [experimental Octopus Pipe](https://bitbucket.org/octopusdeploy/octopus-cli-run/src/master/README.md) for more details on how you can use Bitbucket and Octopus together.
