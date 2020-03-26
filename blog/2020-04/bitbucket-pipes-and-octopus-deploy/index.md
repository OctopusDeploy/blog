---
title: Bitbucket Pipelines - Pipes and integrating with Octopus Deploy
description: What are Bitbucket Pipes and how can you integrate them in your Bitbucket pipeline with Octopus Deploy? This post covers what pipes are and how to integrate Bitbucket pipelines with Octopus.
author: mark.harrison@octopus.com
visibility: private
published: 2020-04-13
metaImage: bitbucket-cd.png
bannerImage: bitbucket-cd.png
tags:
 - DevOps
---

![Bitbucket Pipelines](bitbucket-cd.png)

When I first started at Octopus Deploy, I never really knew much about [Bitbucket](https://bitbucket.org/product). I naively assumed it was just Atlassian's version of Source control. As I found out recently, it's much more than that, and offers (amongst other things) a cloud-based continuous integration solution - [Bitbucket Pipelines](https://bitbucket.org/product/features/pipelines). 

We've written previously about Bitbucket pipelines and how they can help streamline your continous delivery process through the use of [containers](https://confluence.atlassian.com/bitbucket/use-docker-images-as-build-environments-792298897.html). If you aren't familiar with Bitbucket pipelines, I'd recommend having a read of the posts:

 - How to guide - [Setting up a CI/CD pipeline with Bitbucket and Octopus](https://octopus.com/blog/continuous-delivery-bitbucket-pipelines)
 - Container redux - [Using the Octopus CLI containers in your pipeline](https://octopus.com/blog/bitbucket-pipelines-redux)
 
 As part of the Customer Success team, we get to talk to our customers and understand how they are using Octopus. Over time, that experience has taught us that when someone has a particular problem to solve, having a concrete sample to show is helpful for everyone. 
 
 In this post, I'll describe my journey to do just that, with the next evolution in Bitbucket pipelines; [Pipes](https://bitbucket.org/product/features/pipelines/integrations). I create a pipe for an [Octopus CLI](https://g.octopushq.com/OctopusCLI) command and integrate this into a Bitbucket pipeline for our Sample node.js application - [RandomQuotes-Js](https://bitbucket.org/octopussamples/randomquotes-js), and finally deploying it using Octopus.

<h2>In this post</h2>

!toc

## What are Bitbucket Pipes?

Before we dive right in, its important to understand what pipes are, and how they fit into a Bitbucket pipeline. 

Atlassian [says](https://confluence.atlassian.com/bitbucket/learn-about-pipes-978200267.html):

> Pipes provide a simple way to configure a pipeline. They are especially powerful when you want to work with third-party tools. Just paste the pipe into the yaml file, supply a few key pieces of information, and the rest is done for you. You can add as many pipes as you like to your steps, so the possibilities are endless!

Pipes build on the core concept of pipelines; containers. A pipe makes use of a script that lives inside of a [Docker](https://www.docker.com/) container. It typically has the commands that you'd have written in your pipeline yaml file before you used a pipe.

### Example Pipe usage

This is what the Atlassian [bitbucket-upload-file](https://bitbucket.org/product/features/pipelines/integrations?p=atlassian/bitbucket-upload-file) pipe would look like in your pipeline yaml file:

![Bitbucket upload file pipe](bitbucket-upload-file-pipe.png)

where: 
 - `atlassian/bitbucket-upload-file:0.1.3` is the name of the Docker [image](https://hub.docker.com/r/bitbucketpipelines/bitbucket-upload-file) containing the pipe to run.
 - `BITBUCKET_USERNAME` is an example of a variable that you would to provide a value for the pipe to use when executing inside of the container.

 :::hint
**Referring to a Pipe in a pipeline step:** 

There are 2 ways you can refer to a pipe in a step within a pipeline:

1. Refer to the Docker image directly:
```yaml
pipe: docker://<Docker_Account_Name>/<Image_Name>:<tag>
```

2. Refer to a pipe repository hosted on Bitbucket:
```yaml
pipe: <Bitbucket_account>/<Bitbucket_repo>:<tag>
```

This method looks for the location of the Docker image from the `pipe.yml` file within the referenced `<Bitbucket_account>/<Bitbucket_repo>` pipe repository.
 :::

### Why are they useful?

So why go to the trouble of writing a pipe at all? 

Well, Pipes are all about **re-use**. They allow you to repeat the same action in multiple steps of your pipeline.  By centralising the core of your action into a pipe, you end up with a simpler pipeline configuration. Another key feature of a pipe versus directly scripting in your pipeline is the ability to include dependencies that your main pipeline doesn't require.

## Creating a Bitbucket pipe

 A pipe consists of a bunch of files which make up your Docker image. The pipe I created has it's image based on the pre-existing [octopusdeploy/octo](https://hub.docker.com/r/octopusdeploy/octo) image. The finished pipe has also been published as [octopipes/pack](https://hub.docker.com/r/octopipes/pack/) on Docker Hub.

 Firstly, a quick word of warning, creating a pipe can be quite involved. Thankfully, Atlassian provides a step-by-step [guide](https://confluence.atlassian.com/bitbucket/how-to-write-a-pipe-for-bitbucket-pipelines-966051288.html) on their website. 
  
:::warning
I created this pipe on an Ubuntu machine using a bash terminal. If you are using a different platform, you may need to tweak the commands used here.
:::

### Choosing a candidate for a pipe

I've often heard people say that naming something is the hardest thing when it comes to Software, and the same was true for choosing a command to wrap up in a pipe. However in most CI/CD pipelines, once you have built and run any tests on your code, you probably want to package up your applications. So it felt pretty natural to choose the Octopus CLI [pack](https://octopus.com/docs/octopus-rest-api/octopus-cli/pack) command to create a pipe for.

The added bonus was that the `pack` command only has a few required parameters, and the optional ones could be tackled later.

There are 2 types of pipe that you can author:
 - Simple
 - Complete

I opted for a **Complete** pipe so that I could publish it and make use of it in other repositories.
 
### Creating the pipe repository

Next up, I needed to create a new [octopusdeploy/pack](https://bitbucket.org/octopusdeploy/pack) git repository in Bitbucket, and then I cloned it locally.

:::hint
For further information on creating a new Git repository, please see the Atlassian [documentation](https://confluence.atlassian.com/bitbucket/create-a-git-repository-759857290.html).
:::

### Create pipe skeleton

Atlassian provides a method to generate a skeleton of a pipe repository using [Yeoman](http://yeoman.io/). Once you have all the pre-requisites (`nodejs` and `npm`) installed, from a terminal window you can run the generator using the `yo` command:

```bash
yo bitbucket-pipe
```
This will prompt you to select which pipe you want to create. I chose the **New Advanced Pipe (Bash)** option.

![Bitbucket pipe generator -welcome](pipe-generator-welcome.png)

You will be asked some questions to answer - this is all to help fill in the metadata and other useful information for consumers of the pipe. Once complete, it will generate the required files you need to get started:

![Bitbucket pipe generator - complete](pipe-generator-complete.png)

As I did, you'll likely want to edit the following ones to suit your pipe requirements:

 - [pipe.yml](#creating-the-pipes-metadata)
 - [pipe/pipe.sh](#creating-the-pipe-script)
 - [Dockerfile](#creating-the-pipe-Dockerfile)
 - [bitbucket-pipelines.yml](#creating-the-pipes-own-pipeline)
 - [README.md](#creating-the-pipe-readme)

:::hint
**Tip:** Check other repositories to see how they have written their pipe!

One of the great things about every Bitbucket pipe is that the code is public, so you can browse it. For example you can view the source code for the `bitbucket-upload-file` pipe on [Bitbucket](https://bitbucket.org/atlassian/bitbucket-upload-file/).

This is a really great way to see how other authors have structured their pipes.
:::

### Creating the pipe's metadata

When creating a **Complete** pipe, Atlassian requires you to create a `pipe.yml` file. This document contains metadata about your pipe. It includes things like:
 - A friendly name of the pipe.
 - The Dockerhub image for your pipe in the format: `account/repo:tag`.
 - A list of pipe variables where you can specify default values.

If you chose one of the **Advanced** pipes using the pipe generator, then the `pipe.yml` file will be created for you with all of the relevant information already supplied. Here are the contents of my auto-generated [pipe.yml](https://bitbucket.org/octopusdeploy/pack/src/master/pipe.yml) file:

```yaml
name: Octo Pack
image: octopipes/pack:0.0.0
description: Creates a package (.nupkg or .zip) from files on disk, without needing a .nuspec or .csproj
repository: https://bitbucket.org/octopusdeploy/pack
maintainer: support@octopus.com
tags:
    - octopus
    - package
    - deployment
```

### Creating the pipe script

The main part of your pipe is the script or binary which will run when it's executed within a container. It will include all of the logic needed to execute the pipe task. You can choose any language you are familiar with. When I created our [skeleton](#create-pipe-skeleton) of our pipe earlier, I chose to use [bash](https://en.wikipedia.org/wiki/Bash_(Unix_shell)), and a sample `pipe/pipe.sh` file was created for me to finish.

Now I should point out that bash is not *usually* my first choice when faced with a programming challenge. With that being said, I've recently switched my work laptop to [Ubuntu 18.04.4](http://releases.ubuntu.com/18.04.4/) so I felt comfortable writing a pipe with it. 

:::hint
There are some great bash resources online:
 - GNU Manual - [Bash Reference Guide](https://www.gnu.org/software/bash/manual/bash.html)
 - Bash scripting - [A cheatsheet](https://devhints.io/bash)
 - Bash tips and tricks - [Bash one-liners](http://www.bashoneliners.com/)
:::

**TL;DR**

If you want to see the complete pipe script, skip straight to the [end](#complete-pipe-script) or view the [source code](https://bitbucket.org/octopusdeploy/pack/src/master/pipe/pipe.sh). If not, read on!

The general structure to a pipe script file tends to follow this convention:

![pipe script flow](bitbucket-pipe-script-flow.png)

#### Mandatory Pipe variables

For the `pack` command there are 5 parameters I wanted the pipe to handle:

1. The `--id` of the package to create.
1. The `--format` for the package; e.g. `NuPkg` or `Zip`.
1. The `--version` for the package (SemVer).
1. The `--basePath` to specify the root folder containing files and folders to pack.
1. The `--outFolder` to specify the folder where the generated package would be written.

To cater for each of these, I mapped each one to a Bitbucket pipeline [variable](https://confluence.atlassian.com/bitbucket/variables-in-pipelines-794502608.html).

I also followed the validation of variables, shown in the Atlassian [demo-pipe-bash](https://bitbucket.org/atlassian/demo-pipe-bash/src/master/pipe/pipe.sh) script:

```bash
NAME=${NAME:?'NAME variable missing.'}
```
This simply checks for a `$NAME` variable value, and errors with a message when the variable isn't present.

Therefore, for the 5 variables I created, my variable validation looked like this:

```bash
# mandatory parameters
ID=${ID:?'ID variable missing.'}
FORMAT=${FORMAT:?'FORMAT variable missing.'} 
VERSION=${VERSION:?'VERSION variable missing.'}
SOURCE_PATH=${SOURCE_PATH:?'SOURCE_PATH variable missing.'}
OUTPUT_PATH=${OUTPUT_PATH:?'OUTPUT_PATH variable missing.'}
```

#### Optional pipe variables

Next up were some optional variables consumers of the pipe could choose to supply if they wished:

I included an `EXTRA_ARGS` array variable to include multiple additional arguments for the `pack` command. You can specify this variable by using a special Bitbucket Array type in your pipeline:

```yaml
variables:
  EXTRA_ARGS: ['--description', 'text containing spaces', '--verbose']
```

The Array type is really useful, as it allows an easy way to supply any other arguments to the pipe. In my case, this allowed consumers of the `pack` pipe the ability to provide any of the additional argument options that I hadn't explicitly handled.

Lastly, I included a boolean `DEBUG` variable to include additional debugging information. You would specify it's value in your pipe like this:

:::hint
**Advanced Pipe Writing Techniques:** To learn about more the Array type and how its values are passed to the Docker container, please see Atlassian's [documentation](https://confluence.atlassian.com/bitbucket/advanced-techniques-for-writing-pipes-969511009.html).
:::

```yaml
variables:
  DEBUG: 'true'
```

#### Running Octo Pack

The main part of the script is the 


#### Complete pipe script

Here is the finished `pipe.sh` file:

```bash
#!/usr/bin/env bash

# Creates a package (.nupkg or .zip) from files on disk, without needing a .nuspec or .csproj
#
# Required globals:
#   ID
#   FORMAT
#   VERSION
#   SOURCE_PATH
#   OUTPUT_PATH
#
# Optional globals:
#   EXTRA_ARGS
#   DEBUG

source "$(dirname "$0")/common.sh"

# mandatory parameters
ID=${ID:?'ID variable missing.'}
FORMAT=${FORMAT:?'FORMAT variable missing.'} 
VERSION=${VERSION:?'VERSION variable missing.'}
SOURCE_PATH=${SOURCE_PATH:?'SOURCE_PATH variable missing.'}
OUTPUT_PATH=${OUTPUT_PATH:?'OUTPUT_PATH variable missing.'}

FORMAT=$(echo "$FORMAT" | tr '[:upper:]' '[:lower:]')

# Default parameters
EXTRA_ARGS_COUNT=${EXTRA_ARGS_COUNT:="0"}
DEBUG=${DEBUG:="false"}

enable_debug

if [ "${EXTRA_ARGS_COUNT}" -eq 0 ]; then
  # Flatten array of extra args
  debug "Setting EXTRA_ARGS to empty array"
  EXTRA_ARGS=
fi

debug "Flattening EXTRA_ARGS"
init_array_var 'EXTRA_ARGS'

debug ID: "${ID}"
debug FORMAT: "${FORMAT}"
debug VERSION: "${VERSION}"
debug SOURCE_PATH: "${SOURCE_PATH}"
debug OUTPUT_PATH: "${OUTPUT_PATH}"
debug EXTRA_ARGS_COUNT: "${EXTRA_ARGS_COUNT}"
debug EXTRA_ARGS: "${EXTRA_ARGS}"

run octo pack --id "$ID" --version "$VERSION" --format "$FORMAT" --basePath "$SOURCE_PATH" --outFolder "$OUTPUT_PATH" "${EXTRA_ARGS[@]}"

if [ "${status}" -eq 0 ]; then
  OCTO_PACK_FILENAME="$ID.$VERSION.$FORMAT"
  success "Packaging successful. Created package $OUTPUT_PATH/$OCTO_PACK_FILENAME."

else
  fail "Packaging failed."
fi
```

### Creating the pipe Dockerfile


### Creating the pipe's own pipeline

When you have completed your pipe, In order to have the pipe automatically build and deploy new versions of it's container to Docker when you make changes, it's not surprising that you can use Bitbucket pipelines to do just that!

### Creating the pipe README


## Testing the pipe

 - [BATS](https://www.systutorials.com/docs/linux/man/1-bats/)

## Publishing the pipe

 - Dockerfile
 - bitbucket-pipelines.yml

## Integrating the pipe into a pipeline

## Integrating the pipeline with Octopus

## Conclusion

Once I'd got to grips with writing bash, creating my first Bitbucket pipe was pretty straight forward. I can definitely see the advantages of creating a pipe in Bitbucket. That being said, it's important to point out that your pipe shouldnt try to do too much. It's tempting to try to cram as much as you can into a pipe. By doing this you end up fighting against the single biggest advantage that pipes offer; re-use.

## Learn more
  - Take a peek at our first *experimental* pipe - [pack](https://bitbucket.org/octopusdeploy/pack/src/master/README.md)
 - View the sample `bitbucket-pipelines.yml` for [RandomQuotes-Js](https://bitbucket.org/octopussamples/randomquotes-js/src/master/bitbucket-pipelines.yml)
 - Take a look at the Octopus [sample](https://samples.octopus.app/app#/Spaces-104/projects/randomquotes-js) project.
 - Guides - [Octopus CI/CD pipeline Guides](https://octopus.com/docs/guides)

Feel free to leave a comment, and let us know what you think about Bitbucket pipes, pipelines or container-based build chains!