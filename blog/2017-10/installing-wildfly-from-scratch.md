---
title: Installing WildFly From Scratch
description: Learn the steps you'll need to configure a working instance of WildFly.
author: matthew.casperson@octopus.com
visibility: private
metaImage: java-octopus-meta.png
bannerImage: java-octopus.png
tags:
 - Java
---

WildFly and Red Hat JBoss Enterprise Application Platform are [amongst the most popular Java EE application servers available today](https://www.jetbrains.com/research/devecosystem-2017/java/). WildFly is available for free for development and production purposes, and this blog post takes a look at steps required to get WildFly up and running.

## Relationship Between WildFly and JBoss EAP

WildFly and Red Hat JBoss Enterprise Application Platform (JBoss EAP for short) are both open source Java EE application servers.

WildFly is made freely available with community support from the [WildFly website](http://wildfly.org/). WildFly releases major updates quite frequently, and old releases have a short support window.

JBoss EAP is made available as part of a subscription with Red Hat. JBoss EAP is based on the same technology that goes into WildFly, although there is not a one-to-one mapping between JBoss EAP versions and WildFly versions. JBoss EAP releases have much longer support windows than WildFly, and are supported by Red Hat.

:::hint
I have not been able to find any official documentation around the support window for WildFly, although [this forum post](https://developer.jboss.org/thread/267585) does indicate that only the current major release is "supported", with all previous versions being considered legacy releases.
:::

## Java EE Terminology

Java 2 Platform, Enterprise Edition (J2EE) was introduced in 1999 with J2EE 1.2, and the J2EE name was used until 2003 with J2EE 1.4.

In 2005 the name was changed to Java Enterprise Edition (Java EE) with the release of Java EE 5. The Java EE name has been used until 2017 with Java EE 8.

In 2017 Oracle open sourced the Java EE platform, and it is now managed by the [Eclipse Foundation](https://projects.eclipse.org/projects/ee4j/faq) under the name Eclipse Enterprise for Java (EE4J).

## Servlet-Only vs Java EE

WildFly is made available as a Servlet-Only or full Java EE application server distribution.

The Serlvet-Only distribution is a good choice for those looking to deploy applications that don't have any dependency on the Java EE standards. For example, applications written with the Spring library only require a servlet application server to run.

The full application server distribution includes support for the complete Java EE specification, and is a superset of the Servlet-Only distribution.

:::hint
Older Java EE app servers incurred a memory cost from loading libraries that may not have been used by any deployed application, but these days WildFly does a good job of only loading libraries that are required. So while the full application server distribution is a larger package, this may not necessarily mean that it consumes more memory at run time.
:::

For this blog post we'll be using the full application server distribution.
