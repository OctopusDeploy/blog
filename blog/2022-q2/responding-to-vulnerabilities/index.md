---
title: Implementing DevSecOps to respond to vulnerabilities
description: Learn how to configure your CI/CD pipeline to quickly identify and respond to vulnerabilities
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

The Log4j project found itself the centre of attention in late 2021 for a [critical vulnerability](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-45046), spawning a flood of "Our response to Log4j" announcements from companies and framework developers around the globe, and even more requests to support desks and forums from customers looking to gauge their exposure.

Needless to say, many engineering teams found themselves under pressure to identify any code bases that may have been affected, provide accurate reports describing any exposure, update any required dependencies, and deploy the new code to production as quickly as possible.

Anyone facing that challenge knows all too well that such a response is not as easy as it sounds. Like any code dependency, knowing whether your code uses Log4j requires a deep understanding of your application's structure and currently deployed version. Typically this requires checking your code base at the specific git commit representing the deployed version of your application and digging into your direct dependencies, along with their child dependencies, to find exactly what libraries your code uses.

Runbooks, combined with [build information](https://octopus.com/docs/packaging-applications/build-servers/build-information) and some simple changes to your CI/CD pipeline, provide a convenient method for querying the dependencies included in your deployed application. In this post you'll learn how to modify a GitHub Actions Workflow to expose the required information and see an example runbook that can query the information on demand.

## Prerequisites

This post uses GitHub Actions as a CI server. GitHub Actions are free for public git repositories, so you only need a GitHub account to get started.

The sample runbook script is written against Python 3, which can be downloaded from the [Python website](https://www.python.org/downloads/).

The example application and associated GitHub workflows can be found on [GitHub](https://github.com/OctopusSamples/OctoPub).

## Capturing dependencies during the build process

Every major language today provides the ability to list the dependencies consumed as part of the build process. The list below shows examples of these commands, capturing the output in a file called `dependencies.txt`:

* Maven - `mvn --batch-mode dependency:tree --no-transfer-progress > dependencies.txt`
* Gradle - `gradle dependencies --console=plain > dependencies.txt`
* Npm - `npm list --all > dependencies.txt`
* PHP - `composer show --all > dependencies.txt`
* Python - `pip install pipdeptree; pipdeptree > dependencies.txt`
* Go - `go list > dependencies.txt`
* Ruby - `gem dep > dependencies.txt`
* DotNET Core - `dotnet list package > dependencies.txt`

Two steps must be added to a GitHub Actions workflow to capture the dependencies as a artifact. The example below demonstrates how to capture Maven dependencies, but the `run` property of the `List Dependencies` step can be replaced with any of the commands above for your specific use case:

```yaml
    - name: List Dependencies
      run: mvn --batch-mode dependency:tree --no-transfer-progress > dependencies.txt
      shell: bash
    - name: Collect Dependencies
      uses: actions/upload-artifact@v2
      with:
        name: Dependencies
        path: dependencies.txt
```

The screenshot below shows the artifact associated with the build:

![Dependencies Artifact](dependencies-artifact.png "width=500")

## Producing build information

a