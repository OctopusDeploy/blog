---
title: Using the HTTPd Docker image
description: A detailed look at how to use the HTTPd web server with Docker
author: matthew.casperson@octopus.com
visibility: public
published: 2022-07-18-1400
metaImage: blogimage-gettingstartedcontainerisation-2022.png
bannerImage: blogimage-gettingstartedcontainerisation-2022.png
bannerImageAlt: Man sitting on top of  container with green circle with a power up icon
isFeatured: false
tags: 
  - DevOps
  - Containers
  - Cloud Orchestration
---
 
The Apache HTTP Server is one of the most popular web servers available today, with [Wikipedia](https://en.wikipedia.org/wiki/Apache_HTTP_Server) reporting it is either the most used, or second most used, web server in March 2022. An [official HTTPd Docker image](https://hub.docker.com/_/httpd) is made available on Docker Hub and has been downloaded over one billion times, making it one of the most popular Docker images.

In this post I'll show you how to get started with the HTTPd image to host your own web sites or build custom Docker images that embed HTTPd.

## Getting started

The easiest way to use the HTTPd image is to have it host static we content from your local workstation. Save the following HTML code to a file called `index.html`:

```html
<html>
    <body>
        Hello from Octopus!
    </body>
</html>
```

Then run the HTTPd Docker image with the `index.html` file mounted under `/usr/local/apache2/htdocs/index.html` in the container:

```bash
docker run -p 8080:80 -v "$PWD/index.html":/usr/local/apache2/htdocs/index.html httpd:2.4
```

We can then open [http://localhost:8080/](http://localhost:8080/) to view the web page.

Mounting files in this way requires the individual web assets be packaged and distributed to any new server, which is inconvenient. A better solution is to build a custom Docker image that embeds the static web files.

## Creating custom images based on HTTPd

To create your own Docker image, save the following text to a file called `Dockerfile`:

```dockerfile
FROM httpd:2.4
COPY index.html /usr/local/apache2/htdocs/index.html
```

`Dockerfile` contains instructions for building a custom Docker image. Here you use the `FROM` command to base your image on the NGINX one, and then use the `COPY` command to copy your `index.html` file into the new image under the `/usr/local/apache2/htdocs` directory.

Build the new image with the command:

```bash
docker build . -t myhttpd
```

This builds a new image called `myhttpd`. Run the new image with the command:

```bash
docker run -p 8080:80 myhttpd
```

Note that you didn't mount any directories this time. However, when you open `http://localhost:8080/index.html` your custom HTML page is displayed because it was embedded in your custom image.

HTTPd is capable of far more than hosting static web sites. To unlock the full potential of HTTPd we must add custom configuration files.

## Advanced HTTPd configuration

The standard configuration file embedded in the HTTPd image is extracted with the following command:

```bash
docker run --rm httpd:2.4 cat /usr/local/apache2/conf/httpd.conf > my-httpd.conf
```

The resulting file is huge at over 500 lines of code, so we won't list it here.

To have HTTPd load additional configuration files, we add a single line of code to the end of this file which instructs HTTPd to load files in the specified directory: 

```
IncludeOptional conf/sites/*.conf
```

Create a file called `health-check.conf` with the following contents. This configuration enables HTTPd to listen on port 90 and respond to requests on the `/health` path with a 200 OK response. This response simulates a health check of the web server that clients can use to determine if the server is up and running:

```
LoadModule rewrite_module modules/mod_rewrite.so
Listen 90

<VirtualHost *:90>
  <Location /health>
      ErrorDocument 200 "Healthy"
      RewriteEngine On
      RewriteRule .* - [R=200]
  </Location>
</VirtualHost>
```

The `DockerFile` is then updated to create the directory `/usr/local/apache2/conf/sites/`, copy the `health-check.conf` file to the directory, and overwrite the original configuration file with our copy that includes the `IncludeOptional` directive:

```dockerfile
FROM httpd:2.4
RUN mkdir -p /usr/local/apache2/conf/sites/
COPY health-check.conf /usr/local/apache2/conf/sites/health-check.conf
COPY my-httpd.conf /usr/local/apache2/conf/httpd.conf
```

Build the new image with the command:

```bash
docker build . -t myhttpd
```

Run the Docker image with the command. Note that we expose a new port to allow us to access the health check endpoint:

```bash
docker run -p 8080:80 -p 9090:90 myhttpd
```

Then open [http://localhost:9090/health](http://localhost:9090/health) to access the health check page.

## Choosing HTTPd variants

HTTPd images are provided based on Debian or Alpine. Alpine is frequently used as a lightweight base for Docker images. To view the sizes of Docker images, they must first be pulled down to your local workstation:

```bash
docker pull httpd:2.4
docker pull httpd:2.4-alpine
```

You can then find the image sizes with the command:

```bash
docker image ls
```

From this you can see the Alpine variants are much smaller in size, with the Debian based imaged weighing in at 145MB and the Alpine image weighing 55MB.

To use the Alpine variant, base your custom image on the `httpd:2.4-alpine` image:

```dockerfile
FROM httpd:2.4-alpine
RUN mkdir -p /usr/local/apache2/conf/sites/
COPY health-check.conf /usr/local/apache2/conf/sites/health-check.conf
COPY my-httpd.conf /usr/local/apache2/conf/httpd.conf
```

## Conclusion

HTTPd is a popular web server, and the official NGINX Docker image allows DevOps teams to host custom web applications in Docker. With a few small tweaks to the HTTPd configuration files it is also possible to use the full range of [HTTPd modules](https://httpd.apache.org/docs/current/mod/), unlocking many advanced features.

In this post, you learned how to create a custom Docker image hosting a static web application, added advanced HTTPd configuration files to provide a health check endpoint, and compared the sizes of Debian and Alpine NGINX images.

## Resources

* [Octopus trial](https://octopus.com/start)
* [HTTPd Docker Image](https://hub.docker.com/_/httpd)
* [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)

## Learn more

If you'd like to build and deploy containerized applications to AWS platforms such as EKS and ECS, try the [Octopus Workflow Builder](https://octopusworkflowbuilder.octopus.com/#/). The Builder populates a GitHub repository with a sample application built with GitHub Actions workflows and configures a hosted Octopus instance with sample deployment projects demonstrating best practices such as vulnerability scanning and Infrastructure as Code (IaC). 

Happy deployments! 