---
title: Using the NGINX Docker image
description: Learn how to create Docker web apps based on the NGINX image.
author: matthew.casperson@octopus.com
visibility: public
published: 2022-08-09-1400
metaImage: blogimage-gettingstartedcontainerisation-2022.png
bannerImage: blogimage-gettingstartedcontainerisation-2022.png
bannerImageAlt: Man sitting on top of container with green circle with a power up icon
isFeatured: false
tags: 
  - Containers
  - Docker
---

Docker is a compelling platform to package and run web applications, especially when paired with one of the many Platform as a Service (PaaS) offerings provided by cloud platforms. NGINX has long provided DevOps teams with the ability to host web applications on Linux, and also provides an official Docker image to be used as the base for custom web applications. 

In this post I'll explain how DevOps teams can use the [NGINX Docker image](https://hub.docker.com/_/nginx) to build and run web applications on Docker.

## Getting started with the base image

NGINX is a versatile tool with many uses including a load balancer, reverse proxy, and network cache. However, when running NGINX in a Docker container, most of these high level functions are delegated to other specialized platforms or other instances of NGINX. Typically, NGINX fulfils the function of a web server when running in a Docker container.

To create an NGINX container with the default web site, run the following command:

```bash
docker run -p 8080:80 nginx
```

This command will download the `nginx` image (if it has not already been downloaded) and create a container exposing port 80 in the container to port 8080 on the host machine. We can then open [http://localhost:8080/index.html](http://localhost:8080/index.html) to view the default "Welcome to nginx!" web site.

To allow the NGINX container to expose custom web assets, we can mount a local directory inside the Docker container. 

Save the following HTML code to a file called `index.html`:

```html
<html>
    <body>
        Hello from Octopus!
    </body>
</html>
```

Then run the following command to mount the current directory under `/usr/share/nginx/html` inside the NGINX container with read only access:

```bash
docker run -v $(pwd):/usr/share/nginx/html:ro -p 8080:80 nginx
```

Open [http://localhost:8080/index.html](http://localhost:8080/index.html) again and we will see the custom HTML page displayed.

One of the benefits of Docker images is the ability to bundle all related files into a single distributable artifact. To realize this benefit we must create a new Docker image based on the NGINX image.

## Creating custom images based on NGINX

To create our own Docker image, save the following text to a file called `Dockerfile`:

```dockerfile
FROM nginx
COPY index.html /usr/share/nginx/html/index.html
```

`Dockerfile` contains instructions for building a custom Docker image. Here we use the `FROM` command to base our image on the NGINX one, and then use the `COPY` command to copy our `index.html` file into the new image under the `/usr/share/nginx/html` directory.

Build the new image with the command:

```bash
docker build . -t mynginx
```

This builds a new image called `mynginx`. Run the new image with the command:

```bash
docker run -p 8080:80 mynginx
```

Note that we did not mount any directories this time. However, when we open [http://localhost:8080/index.html](http://localhost:8080/index.html) our custom HTML page is displayed because it was embedded in our custom image.

NGINX is capable of much more than hosting static files. To unlock this functionality, we must use custom NGINX configuration files.

## Advanced NGINX configuration

NGINX exposes its functionality via configuration files. The default NGINX image comes with a simple default configuration file designed to host static web content. This file is located at `/etc/nginx/nginx.conf` in the default image, and has the following contents:

```
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```

There is no need to understand this configuration file in detail, but there is one line of interest that instructs NGINX to load additional configuration files from the `/etc/nginx/conf.d` directory:

```
include /etc/nginx/conf.d/*.conf;
```

The default `/etc/nginx/conf.d` file configures NGINX to function as a web server. Specifically the `location /` block loading files from `/usr/share/nginx/html` is why we mounted our HTML files to that directory previously:

```
server {
    listen       80;
    server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}

```

We can take advantage of the instruction to load any `*.conf` configuration files in `/etc/nginx` to customize NGINX. In this example we'll add a health check via a custom location listening on port 90 that responds to requests to the `/nginx-health` path with a HTTP 200 OK.

Save the following text to a file called `health-check.conf`:

```
server {
    listen       90;
    server_name  localhost;

    location /nginx-health {
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }
}
```

Modify the `Dockerfile` to copy the configuration file to `/etc/nginx/conf.d`:

```Dockerfile
FROM nginx
COPY index.html /usr/share/nginx/html/index.html
COPY health-check.conf /etc/nginx/conf.d/health-check.conf
```

Build the image with the command:

```bash
docker build . -t mynginx
```

Run the new image with the command. Note the new port exposed on 9090:

```bash
docker run -p 8080:80 -p 9090:90 mynginx
```

Now open [http://localhost:9090/nginx-health](http://localhost:9090/nginx-health). The health check response is returned to indicate that the web server is up and running.

The examples above base our custom images on the default `nginx` image. But there are other variants that provide much smaller image sizes without sacrificing any functionality.

## Choosing NGINX variants

The default `nginx` image is based on [Debian](https://github.com/nginxinc/docker-nginx/blob/master/Dockerfile-debian.template). However, NGINX also provides images based on [Alpine](https://github.com/nginxinc/docker-nginx/blob/master/Dockerfile-alpine.template).

Alpine is frequently used as a lightweight base for Docker images. To view the sizes of Docker images, they must first be pulled down to our local workstation:

```bash
docker pull nginx
docker pull nginx:stable-alpine
```

We can then find the image sizes with the command:

```bash
docker image ls
```

From this we can see the Debian image weighs around 140 MB while the Alpine image weights around 24 MB.

To base our images on the Alpine variant, we need to update the `Dockerfile`:

```Dockerfile
FROM nginx:stable-alpine
COPY index.html /usr/share/nginx/html/index.html
COPY health-check.conf /etc/nginx/conf.d/health-check.conf
```

Build the and run image with the commands:

```bash
docker build . -t mynginx
docker run -p 8080:80 -p 9090:90 mynginx
```

Once again open [http://localhost:9090/nginx-health](http://localhost:9090/nginx-health) or [http://localhost:8080/index.html](http://localhost:8080/index.html) to view the web pages. Everything continues to work as it did, but our custom image is now much smaller.

## Conclusion

NGINX is a powerful web server, and the official NGINX Docker image provides DevOps teams with the ability to host custom web applications in Docker. NGINX also supports advanced scenarios thanks to its ability to read configuration files copied into a custom Docker image.

## Resources

* [Octopus trial](https://octopus.com/start)
* [NGINX Docker Image](https://hub.docker.com/_/nginx)
* [NGINX Docker Image Source Code](https://github.com/nginxinc/docker-nginx)
* [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)

## Learn more

If you're looking to build and deploy containerized applications to AWS platforms such as EKS and ECS, the [Octopus Workflow Builder](https://octopusworkflowbuilder.octopus.com/#/) populates a GitHub repository with a sample application built with GitHub Actions workflows and configures a hosted Octopus instance with sample deployment projects demonstrating best practices such as vulnerability scanning and Infrastructure as Code (IaC). 

Happy deployments! 
