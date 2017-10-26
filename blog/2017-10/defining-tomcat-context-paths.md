---
title: Defining Tomcat Context Paths
description: Learn how Tomcat defines the context path of your web application.
author: matthew.casperson@octopus.com
visibility: private
metaImage: java-octopus-meta.png
bannerImage: java-octopus.png
tags:
 - Java
---

## Exploded Deployments vs WAR Packages

There are two ways to deploy Java web applications.

The first way is to deploy a WAR file. A WAR file is just a ZIP archive with a directory structure that is recognised by Java application servers like Tomcat. WAR files are convenient because they are a single package that is easy to copy, and it compresses its contents making it quite compact.

The second way is to deploy all the individual files that make up a web application. This kind of deployment can be very useful during development, as files like HTML pages and CSS files can be edited while the application is deployed and reloaded on the fly.

:::hint
By default when you deploy a WAR file to Tomcat, it will be extracted into an exploded deployment for you. In the screenshot below you can see that the end result of deploying a file called `demo.war` is a directory called demo with the context of the `demo.war` archive extracted into it.

![Tomcat Exploded Deployment](tomcat-exploded-deployment.png)

This behaviour can be disabled by setting the `unpackWARs` attribute on the `<Host>` element in the `conf/server.xml` configuration file to `false`.
:::

## The webapps Directory

The `webapps` directory is where deployed applications reside in Tomcat.

:::hint
The `webapps` directory is the default deployment location, but this can be configured with the `appBase` attribute on the `<Host>` element in the `conf/server.xml` configuration file.
:::

If Tomcat is set to autodeploy applications (and it is set to do this by default) then any WAR files copied into the `webapps` folder will be deployed automatically.



## Embedding the Path in the WAR Filename

If no other settings are in effect, a web application deployed to Tomcat will be available under a context path that matches the name of the WAR file or the directory under `webapps` that the exploded deployment was copied to.

## Deploying to the Root Context

## Adding a context.xml File

## Uploading via the Manager Application
