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

## Downloading WildFly

WildFly is made available as a zip or tar.gz package. Either is fine for Windows users. For Linux users, the tar.gz package is preferred as it will retain the executable flag for shell scripts.

At the time this blog post was written, WildFly 10.1.0 was the last major release. However, we'll use WildFly 11 CR1 as it is going to be very close to the final WildFly 11 release, which is due any time now.

## Standalone vs Domain

WildFly can be run in two different modes: standalone or domain.

Standalone mode is used when running a WildFly instance that manages its own configuration and deployments.

Domain mode is used to configure and deploy applications to multiple WildFly instances. In Domain mode the Domain Controller distributes configuration and applications to Domain Slaves.

:::hint
A domain should not be confused with a cluster. Domains exist only to distribute settings and applications, and those settings may or may not build a clustered environment. Likewise standalone instances can participate in a cluster if they are individually configured with the required settings.
:::

## Manually Running WildFly

WildFly can be started in either domain or standalone mode. Each mode is launched with a separate script.

To start WildFly in standalone mode, run the `bin\standalone.bat` script in Windows and `bin/standalone.sh` script for Linux.

To start WildFly in domain mode, run the `bin\domain.bat` script in Windows and `bin/domain.sh` script for Linux.

## Configuring WildFly Memory Settings

The memory settings used by WildFly are defined in the `JAVA_OPTS` environment variable. In the absence of this environment variable, default values are defined in the `bin/standalone.conf.bat` and `bin/domain.conf.bat` files for Windows, and the  `bin/standalone.conf` or `bin/domain.conf` files for Linux.

In the Windows configuration files you will see commands like this:

```
if not "x%JAVA_OPTS%" == "x" (
  echo "JAVA_OPTS already set in environment; overriding default settings with values: %JAVA_OPTS%"
  goto JAVA_OPTS_SET
)
rem # ...
set "JAVA_OPTS=-Xms64M -Xmx512M -XX:MetaspaceSize=96M -XX:MaxMetaspaceSize=256m"
```

In the Linux configuration files you will see commands like this:

```
if [ "x$JAVA_OPTS" = "x" ]; then
   JAVA_OPTS="-Xms64m -Xmx512m -XX:MetaspaceSize=96M -XX:MaxMetaspaceSize=256m -Djava.net.preferIPv4Stack=true"
   JAVA_OPTS="$JAVA_OPTS -Djboss.modules.system.pkgs=$JBOSS_MODULES_SYSTEM_PKGS -Djava.awt.headless=true"
else
   echo "JAVA_OPTS already set in environment; overriding default settings with values: $JAVA_OPTS"
fi
```

To override the default values assigned to `JAVA_OPTS` in these configuration files, you can define the `JAVA_OPTS` environment variable. The environment variable takes precedence over the defaults. Otherwise you can edit the default values in the configuration files directly.

You can find more information on the settings shown here by viewing the Oracle documentation. For example, [this documentation](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/java.html) lists the options for Java 8.

:::hint
Some Java settings are version specific. Be sure to use settings that are specific to the version of Java you are running WildFly with.
:::

## Installing WildFly as a Service

Production WildFly instances are typically started as a service. This allows WildFly to be started when the operating system boots, shutdown when the OS is shutdown, and managed with the service management tools built into the OS.

### Installing Tomcat as a Windows Service

### Installing Tomcat as a Linux Service

WildFly ships with init.d and systemd service definition files in the `docs/contrib/scripts/init.d` and `docs/contrib/scripts/systemd` directories.



## Configuring Admin Users

## Opening the Admin Console

## Conclusion
