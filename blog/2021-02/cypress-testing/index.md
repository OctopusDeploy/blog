---
title: End to end testing with Cypress
description: Learn how to run end to end tests as part of an Octopus deployment
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

The idea of testing your code as part of your development process has won almost universal adoption. The canonical example describing the testing patterns performed during an application's lifecycle is the testing pyramid (although there are many alternatives like [the testing honeycomb](https://engineering.atspotify.com/2018/01/11/testing-of-microservices/)) which describe certain strategies like end-to-end tests that often require a live instance of your application to be running in order for the tests to be performed.

One such example of end-to-end testing is via tools like Cypress, which interact with a web page in much the same way a human would. These tests necessarily require the web application to be running, which make them an ideal candidate to be included in the final stages of your deployment process once your web application is deployed and running in a test environment. 

In this blog post we'll look at some of the practical concerns around running Cypress during an Octopus deployment, and present a solution that allows Cypress tests to be run in most common scenarios.

## Including Cypress in your deployment process

The first and most pressing issue to resolve is how to get Cypress into your deployment pipeline.

Perhaps the most obvious way to achieve this is to install Cypress and a web browser onto a VM or physical machine. There are many tools like Ansible, Puppet, and Chef that can be used to automate this process in a repeatable way.

The downside to having the tools directly installed in this way is that you can not easily do it when using dynamic workers in a hosted Octopus instance, and in platforms like Kubernetes there is no concept of a VM for you to preconfigure.

A more generic solution is to [run Cypress from a Docker container](https://github.com/cypress-io/cypress-docker-images). This is convenient because the Cypress team has already done the hard work of configuring all the required software in their DOcker images. You also gain the ability to switch browser versions on a whim simply by running a different Docker image version.

## Referencing Cypress tests

The next issue we need to resolve is how the individual Cypress tests are included in the deployment.

It is possible to bake the tests into a custom Docker image, and if your tests don't tend to change much, this may be a perfectly reasonable solution.

However, I suspect most teams that invest in end-to-end testing will want to retain the ability to quickly update their tests scripts without the burden of then including them in new Docker images. Wouldn't it be nice to have a common, generic Cypress Docker image that could execute random test scripts?

When running Docker directly, this is relatively easy: you simply mount a local directory containing your test scripts into the generic Cypress docker image. The [documentation](https://github.com/cypress-io/cypress-docker-images/tree/master/included) provides an example like this, which mounts the current directory as the `e2e` directory in the Docker container:

```
docker run -it -v $PWD:/e2e -w /e2e cypress/included:6.4.0
```

Unfortunately Kubernetes does not support this kind of volume mounting. [You can mount the contents of a config map as a file](https://stackoverflow.com/questions/33415913/whats-the-best-way-to-share-mount-one-file-into-a-pod), but this option [does not support directory structures](https://github.com/kubernetes/kubernetes/issues/62421). This is a distinct limitation when testing with Cypress, as the [Cypress directory structure includes many subdirectories](https://docs.cypress.io/guides/core-concepts/writing-and-organizing-tests.html#Folder-Structure).

Octopus provides a solution via [worker containers](https://octopus.com/docs/projects/steps/execution-containers-for-workers), which execute a deployment step inside a container generated from an appropriately configured Docker image. The Octopus step can then download and extract a generic package, and run a custom script.

In our case, we will create a custom (but still generic) Docker image based on the Cypress image that Octopus can execute inside of to extract a package containing our test script and execute Cypress.

Our Docker image is built with the following `Dockerfile`:

```
FROM cypress/included:6.4.0
RUN apt-get update; apt-get install -y libicu-dev
RUN npm install -g inline-assets
ENTRYPOINT []
```

This `Dockerfile` is based on the Cypress image `cypress/included:6.4.0`, installs `libicu-dev` (which is required by .NET Core applications under Linux), installs the [inline-assets](https://www.npmjs.com/package/inline-assets) tool to process the Cypress HTML report (more on that later), and resets the `ENTRYPOINT` so Octopus can override the command used when running the image.

This Docker image can be built and published with the following command, replacing `dockerhubusername` with your own Docker Hub user name:

```
docker build . -t dockerhubusername/workerimage
docker push dockerhubusername/workerimage
```

In my case, I created an image called `mcasperson/workerimage`. We can now use this image to create the Octopus execution container.

## Creating the sample Cypress test

For this example we will create a simple introductory Cypress test that performs no real work, but allows us to simulate the process of running end-to-end tests. The code for this sample test can be found on [GitHub](https://github.com/OctopusSamples/simple-cypress-test).