---
title: Using the Alpine Docker image
description: A detailed look at how to use the Alpine Docker image
author: matthew.casperson@octopus.com
visibility: private
published: 2022-09-12-1400
metaImage: blogimage-gettingstartedcontainerisation-2022.png
bannerImage: blogimage-gettingstartedcontainerisation-2022.png
bannerImageAlt: Man sitting on top of  container with green circle with a power up icon
isFeatured: false
tags: 
  - DevOps
  - Containers
  - Cloud Orchestration
  - Docker
---

Thanks to its small size, the [Alpine Docker image](https://hub.docker.com/_/alpine) is frequently used as the base for other custom images. With over a billion downloads on Docker Hub, Alpine is also one of the most popular images available. (You can also read my [post about Ubuntu](https://octopus.com/blog/using-ubuntu-docker-image), which currently claims top spot as most downloaded image from Docker Hub.)


In this post, I show you the best practices to adopt when basing your own images on Alpine.

## Cleaning the cache

Cached package lists are useful when installing software on workstations and servers as they provide quick access to the available packages in a package repository. However, packages installed in Docker containers are rarely updated at runtime; instead the Docker image itself is updated and the container is recreated. This means package cache lists are unnecessary and inefficient to bake into Docker images.

One method to remove cached package lists is to install new packages with the `apk add --update-cache` command and then delete the files under `/var/cache/apk` as part of a single `RUN` instruction. This ensures the package cache is created, which is required to install additional packages, and then cleaned up without capturing the package cache in intermediate image layers:

```dockerfile
RUN apk add --update-cache \
    python \
    python-dev \
    py-pip \
    build-base \
  && pip install virtualenv \
  && rm -rf /var/cache/apk/*
```

You can also use the `apk add --no-cache` option. This is equivalent to the previous command, but it’s more concise:

```dockerfile
RUN apk add --no-cache nginx
```

## Virtual packages

Virtual packages provide a way to bundle packages under a common name, allowing them to be removed as a group. The [Alpine Docker image documentation](https://github.com/alpinelinux/docker-alpine/blob/master/docs/usage.adoc) provides this example where Python development libraries are installed, the dependencies of a Python application are downloaded, and the Python development libraries are then removed:

```dockerfile
FROM alpine

WORKDIR /myapp
COPY . /myapp

RUN apk add --no-cache python py-pip openssl ca-certificates
RUN apk add --no-cache --virtual build-dependencies python-dev build-base wget \
  && pip install -r requirements.txt \
  && python setup.py install \
  && apk del build-dependencies

CMD ["myapp", "start"]
```

While this example works, it's inefficient. Due to the way Docker caching is implemented, any changes to the files copied into the image with the instruction `COPY . /myapp` invalidates the cache, forcing subsequent instructions to be rerun. In practice, this means the example above will download, install, and delete the Python development libraries every time the Python code is changed.

A better solution is to use [multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/). An example is shown below:

```dockerfile
FROM alpine AS compile-image

RUN apk add --no-cache python3 py-pip openssl ca-certificates python3-dev build-base wget

WORKDIR /myapp

COPY requirements.txt /myapp/
RUN python3 -m venv /myapp
RUN /myapp/bin/pip install -r requirements.txt

FROM alpine AS runtime-image

RUN apk add --no-cache python3 openssl ca-certificates

WORKDIR /myapp
COPY . /myapp

COPY --from=compile-image /myapp/ ./

CMD ["/myapp/bin/python", "myapp.py", "start"]
```

A multistage build lets you create an image with development libraries required to build application source code. This "compile image" retains these development libraries between builds, removing the need to download them every time.

A second image is created to host the executable application code and runtime libraries, but specifically does not include any libraries only required at compile time. This "runtime image" is as small as possible as it copies the files produced by the compile image without requiring the associated compile time libraries.

## musl vs glibc

For the most part, Alpine can be used as a drop-in replacement for any other base Docker image. However, it's important to be aware of the architectural differences between Alpine and other common base Docker images such as Ubuntu, Debian, or Fedora.

Alpine uses the [musl C standard library](http://musl.libc.org/), while Ubuntu, Debian, and Fedora use [glibc](https://www.gnu.org/software/libc/). Here is a [detailed comparison of the two libraries](http://www.etalabs.net/compare_libcs.html).

There are some circumstances where third-party tools assume or require glibc. For example, the [Visual Studio Code remote container execution documentation](https://code.visualstudio.com/docs/remote/containers) provides this warning:

> When using Alpine Linux containers, some extensions may not work due to glibc dependencies in native code inside the extension.

Alpine based images are also [not suitable for use as Octopus container images](https://octopus.com/docs/projects/steps/execution-containers-for-workers):

> Linux distributions built on musl, most notably Alpine, do not support Calamari, and cannot be used as a container image. This is due to Calamari currently only being compiled against glibc and not musl.

PythonSpeed's blog post, [Using Alpine can make Python Docker builds 50× slower](https://pythonspeed.com/articles/alpine-docker-python/), details some performance issues when building Python applications on Alpine:

> Most Linux distributions use the GNU version (glibc) of the standard C library that is required by pretty much every C program, including Python. But Alpine Linux uses musl, those binary wheels are compiled against glibc, and therefore Alpine disabled Linux wheel support.

Outside of specific use cases with known incompatibilities with musl, I've found Alpine to be a reliable and practical choice to base my own Docker images on. But it's good to be aware of the implications of using distributions implementing musl.

## Conclusion

Alpine provides a lightweight and popular Docker image that can improve your image build and deployment times compared to other popular images like Ubuntu. 

Alpine uses the musl C standard library, which may introduce compatibility issues in some circumstances, but you can generally assume Alpine provides everything you need for your custom Docker images. Advanced features such as virtual packages may also allow you to keep image sizes down, although multi-stage builds are likely to be a better choice.

## Resources

* [Octopus trial](https://octopus.com/start)
* [Alpine Docker image](https://hub.docker.com/_/alpine)
* [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)
* [Multistage builds](https://docs.docker.com/develop/develop-images/multistage-build/)

## Learn more

If you'd like to build and deploy containerized applications to AWS platforms such as EKS and ECS, try the [Octopus Workflow Builder](https://octopusworkflowbuilder.octopus.com/#/). The Builder populates a GitHub repository with a sample application built with GitHub Actions workflows and configures a hosted Octopus instance with sample deployment projects demonstrating best practices such as vulnerability scanning and Infrastructure as Code (IaC). 

Happy deployments! 