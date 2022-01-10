---
title: What are the benefits of using NGINX Unit?
description: What features does NGINX Unit have that make it an attractive solution for modern deployments?
author: matthew.casperson@octopus.com
visibility: public
published: 2019-11-04
metaImage: nginx-unit.png
bannerImage: nginx-unit.png
bannerImageAlt: NGINX Unit web server
tags:
 - DevOps
 - Kubernetes
---

![NGINX Unit web server](nginx-unit.png)

NGINX is one of the most popular web servers and reverse proxies available today. It offers high performance, almost infinite configurability, and is a commonly used component in modern stacks like Kubernetes. Now the NGINX team has a new offering called NGINX Unit, which is aimed at solving some common challenges of modern development processes.

If you jump to the [NGINX Unit documentation](https://unit.nginx.org/#about), you’ll find a wonderful [live demo from NGINX Conf 17](https://youtu.be/I4IWEz2lBWU). There is also an [overview session](https://www.youtube.com/watch?v=EU78CIR3CeU) from the same conference. These presentations did a good job of answering what NGINX Unit does, but still left me wondering why someone would choose to use NGINX Unit.

In this blog post, we’ll look at some of the advantages provided by NGINX Unit.

## Standardized JSON configuration files

The traditional NGINX configuration files tend to look like this:

```
server {
  location / {
    fastcgi_pass localhost:9000;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_param QUERY_STRING  $query_string;
  }

  location ~ \.(gif|jpg|png)$ {
    root /data/images;
  }
}
```

While this format should be reasonably familiar to developers from curly-quote languages, the NGINX config file syntax does not conform to any general standard. If you want to update this file on a regular basis, you can expect to write complex `sed` commands to edit the raw text.

But using regular expressions to modify configuration files is not a pleasant experience. Inevitably, you will find your regex has matched something you didn’t expect, didn’t account for line endings or didn’t quite capture all the variations of a value.

NGINX Unit addresses this by utilizing JSON for its configuration. There is no longer any ambiguity as to how to structure the configuration data, and it is much easier to update configuration values programmatically. Relying on a common data format makes NGINX Unit much easier to manage.

## HTTP configuration API

Every modern computing platform has a rich CLI tool backed by a well-structured API. It’s easy to take this functionality for granted until you find yourself running `sed` against a configuration file and then restarting a service.

While NGINX Unit doesn’t provide a CLI tool (all the examples in the documentation use `curl`), it does expose all of the configuration via an easy to understand HTTP API. This provides you with a great deal of flexibility in choosing how to expose the API (i.e., expose it on localhost or [securely proxy it to make it publicly available](https://unit.nginx.org/howto/integration/#securely-proxying-unit-api)), and it means you can use any scripting tool of your choice to interact with it.

## Declarative model for common use cases

Given the ubiquity of NGINX, I find myself having to read a random NGINX config file a few times a year. Each time I have to go to the documentation to refresh my memory as to what the commands do, and each time I find myself having to reverse engineer the intent of the configuration from the sequence of commands.

NGINX Unit does away with this imperative configuration model and instead exposes a declarative configuration model. Admittedly, the NGINX Unit model is far less configurable than a traditional NGINX configuration file, but it does a good job of exposing the common routing and security options you will use.

The declarative configuration model makes NGINX Unit far more approachable for those with experience configuring cloud services and platforms like Kubernetes.

## Consistent networking layer

Polyglot development is increasingly popular. Whether it’s because your team has found a mix of languages that best suits their needs or because your infrastructure includes third party products written in multiple languages, it is not uncommon to have an application stack that contains a selection of programming languages.

However, the networking layer still needs to be treated in a cohesive way, which is challenging when every application has a network component that is configured in a slightly different way.

NGINX Unit is part of a growing trend to lift the networking concerns out of the individual applications to make it the responsibility of the infrastructure layer. NGINX Unit consolidates networking concerns by exposing a common API and handling common networking tasks like security and routing.

## Reconfiguration without restarts

NGINX Unit provides a clean separation between the application processes it hosts and the networking layer it places over the top. This makes it possible to change the networking configuration without restarting the hosted applications. It also means that networking changes can be applied to a running system with no downtime.

Combine this ability to apply changes to a live system with the consistent network configuration that all applications hosted by NGINX Unit can take advantage of, and you have an infrastructure that can easily scale up as the number of deployments and application versions increase.

## Low barrier to adoption

NGINX Unit installs in seconds, is started with a single command, and will host your first application with a single update to the configuration. It doesn’t require agents, control planes, databases, or a dedicated cluster to run.

When you compare this to the amount of work that has to go into your average cloud deployment or getting a Kubernetes cluster up and running for the first time, NGINX Unit is a breath of fresh air.

What you see is what you get with NGINX Unit, making it easy to deploy, reason about and diagnose.

## Managing applications as services

How do you manage your production deployments of Node.js applications today? Node.js itself doesn't really provide a solution here, which has spawned a host of solutions like [PM2](https://pm2.keymetrics.io/), [forever](https://www.npmjs.com/package/forever), nohup, or native service management via systemd.

NGINX Unit is a capable alternative for hosting applications that have traditionally relied on third-party solutions to boot, run, and restart processes, with the added benefit that it can host applications written in multiple languages.

## Downsides to using Unit

There are some downsides, or at least issues to be aware of, when using NGINX Unit:

* There is no Windows port, which limits NGINX Unit to Linux and MacOS shops only.
* Go and Node.js codebases have to be modified to work with NGINX Unit.
* NGINX Unit is limited to certificate handling and basic routing. For example, you won’t find authentication, complex rewriting rules, fault injection, retries, weighted traffic splitting, etc.
* NGINX Unit does not provide health checks for the processes it manages.

## Conclusion

Overall, I get the impression that NGINX Unit is what NGINX would look like if it were written today. It offers a configuration model based on JSON, exposes that configuration via an HTTP API, and provides a declarative model for configuring the most common networking use cases, all while being lightweight and simple to run.

This back-to-basics approach does mean that some use cases currently supported by NGINX are not possible in NGINX Unit, but with a [new release every few months](https://unit.nginx.org/CHANGES.txt), I expect the functionality of NGINX Unit will grow.

If the functionality provided by NGINX Unit meets your needs today, it is the natural choice over deploying the traditional NGINX service.
