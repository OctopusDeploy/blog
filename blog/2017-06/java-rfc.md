---
title: "Octopus Deploy Java Support RFC"
visibility: public
author: matthew.casperson@octopus.com
description: "Our plans to support Java as a first class citizen in Octopus Deploy"
published: 2017-06-09
metaImage: java-octopus-meta.png
bannerImage: java-octopus.png
tags:
 - Product
---

Octopus was originally built with .NET developers in mind, and comes with a number of conventions that make deploying .NET applications easy.  At a basic level, Octopus provided Tentacle as a transport layer, and the ability to execute PowerShell.  Layered on top of that, there are built-in steps to deploy an app to IIS, or to set up a Windows service, and there are conventions to deal with .NET configuration files or transforms.

Of course you can already use Octopus for more than just .NET applications - you could ZIP a Java application and push it to a Windows machine, and use PowerShell to configure it. Or more recently you could do the same with SSH and Bash. And that’s enough to make Java deployments work - I recently showed how to deploy to both [WildFly](https://octopus.com/blog/wildfly-deploy) and [Tomcat](https://octopus.com/blog/octopus-tomcat), as well as deploying [Spring Boot applications as Windows services](https://octopus.com/blog/spring-boot-windows-services).

You can make it work, but there aren’t currently any high-level conventions for Java apps. You could say that Octopus currently supports Java apps, but doesn’t have *first-class support* for them, the way it does for .NET apps.

## First-class Java support in Octopus
Today, we’re announcing our long-term vision to make our Java deployment story equal to our .NET deployment story - we want them both to be equal, first-class experiences. Today’s RFC looks at our initial plans for what this means - what we’re planning to build soon - but we’re also looking for direction from you: what does first-class Java support in Octopus mean for you?

## Goals
The goal of the features outlined in this RFC is to enable first class support for deploying Java applications to Tomcat and WildFly / JBoss EAP running on Windows and Linux.

## Application Server Support
Initially Octopus Deploy will support Tomcat and WildFly / JBoss EAP. These application servers were chosen because of their large market share, which when combined represent around two thirds of the Java servers in production.

We will be targeting Red Hat JBoss EAP 6 and above, Wildfly 10 and above, and Tomcat 7 and above.

It is unlikely that we will be able to implement support for all applications servers initially, but if you would like to see support for WebSphere, WebLogic, Glassfish, Payara etc. please add a comment.

Supporting these applications servers means adding the following steps to Octopus Deploy:

* Deploying packages
 * File copying (i.e. copying WAR files to the `webapps` or `deployments` directory, or exploding WAR files)
 * HTTP uploads through the Tomcat manager
 * CLI deployments with the WildFly CLI tool
* Certificate management
 * Exporting JKS Java keystores
 * Configuring a keystore in WildFly or Tomcat
* Creation and deployment of WildFly vaults to store sensitive information that can be referenced in a WildFly configuration file.

## Configuration Support
To support the variety of configuration files found in Java applications, steps will be added to enable transformation of XML, YAML and Properties files in arbitrary locations in Java packages. This will allow Octopus Deploy to support the XML configuration files used extensively by Spring and Java Servlet or JavaEE applications, as well as [Spring externalized configuration](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html).

## Linux Support
As Linux is quite popular with Java developers, the existing step to deploy a service will be extended to support both init and systemd.

Currently we require Mono to be installed on Linux servers. We’ve been moving parts of Calamari and Tentacle to .NET Core, so this should eliminate the Mono dependency.

## Package Support
Currently Octopus Deploy accepts ZIP and Nuget packages. To support Java, the JAR, WAR, EAR and RAR package formats will be natively supported. This will allow Java packages to be uploaded without first wrapping them in a ZIP file.

Octopus will also support Maven repositories as first-class package feeds, the way NuGet and Docker Registry are currently supported.

## Versioning Support
Support for the Maven versioning scheme will be added to allow Octopus Deploy to compare the versions of two Java packages. The version information will be found in the `META-INF/manifest.mf` file under the `Implementation-Version` key.

## Feedback
If you have any suggestions or comments about the changes we’re proposing, please leave a comment. If there are Java deployment scenarios that you would like to see supported by Octopus Deploy, or if you have a specific feature request, your feedback is valuable.
