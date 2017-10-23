---
title: Installing Tomcat From Scratch
description: Learn the steps you'll need after you download and extract Tomcat
author: matthew.casperson@octopus.com
visibility: private
tags:
 - Java
---

Tomcat is the most popular Java web server available today, and is a solid choice for anyone looking to host their Java web applications. One of the nice things about Tomcat is that it is quite easy to get started, often requiring little more than downloading and extracting the deployment archive. But there are a few steps that every Tomcat administrator should know to get the most out of their Tomcat installation.

In this blog post we'll walk through the process of setting up a Tomcat server.

## Install Java

Being a Java web server, Tomcat requires Java to be installed before it can be run.

Tomcat 7 requires at least Java 6, Tomcat 8 requires Java 7, and Tomcat 9 requires Java 8.

Keep in mind that you will need to install a version of Java that supports both Tomcat itself and the applications that Tomcat will host. If your applications are compiled for Java 8, then Tomcat will need to be run with Java 8 as well.

## Download Tomcat

Tomcat can be downloaded from [Apache Tomcat](https://tomcat.apache.org/index.html) homepage.

Tomcat can be downloaded as a zip, tar.gz, Windows zip or Windows exe.

### Download Tomcat for Linux
If you are hosting Tomcat in Linux, then the tar.gz package are what you need. This is preferred over the zip package because the tar.gz format retains the executable flag on startup scripts. If you download the zip package in Linux, you will need to manually set the executable flag on scripts like `bin/startup.sh` and `bin/shutdown.sh` with the command `chmod +x <scriptname>`.

### Download Tomcat for Windows
If you are running in Windows then you can download any of the formats. However, I would recommend that Windows users download either the Windows zip or exe packages.

These packages include the `tcnative-1.dll` library, which is part of the [Tomcat Native](https://tomcat.apache.org/native-doc/) library. Tomcat Native is used to give Tomcat access to the [Apache Portable Runtime](https://apr.apache.org/) (APR). AP in turn is used for features like providing HTTPS via OpenSSL, which can provide much better performance than using the native Java HTTPS implementations (otherwise known as the JSSE implementation).

In addition, the Windows packages also include executables that are used to install Tomcat as a Windows service.

## Configuring the JAVA_HOME Environment Variable

The `JAVA_HOME` environment variable is used by Tomcat scripts to find the Java executable that will be used to launch the Tomcat web server.

### Configuring the JAVA_HOME Environment Variable in Windows

Windows defines environment variables in the system properties.

### Configuring the JAVA_HOME Environment Variable in Linux

There are multiple ways to define environment variables in Linux. The most common is to add the environment variable to the `/etc/environment` file.

## Running Tomcat

To manually launch Tomcat, you will need to run the `bin\startup.bat` batch file for Windows, or the `bin/startup.sh` shell script for Linux.

## Installing Tomcat as a Service

Production Tomcat instances are typically started as service. This allows Tomcat to be started when the operating system boots, shutdown when the OS is shutdown, and managed with the service management tools built into the OS.

### Installing Tomcat as a Windows Service

The easiest way to install Tomcat as a Windows service is to run the `Windows Service Installer` exe, which is one of the Tomcat download package options. This installer provides a wizard that will configure Tomcat as a Windows service.

Alternatively you can use the `bin\service.bat` file to manually configure Tomcat as a Windows service. For example, running the command `service.bat install MyService` will configure Tomcat under a Windows service called `MyService`.

See this [documentation](https://tomcat.apache.org/tomcat-9.0-doc/windows-service-howto.html#Installing_services) for more information on manually configuring a Tomcat Windows service.

:::hint
The `service.bat` file that comes with the versions of Tomcat I tested didn't work well with the Java 9 JDK. I recieved the error `The JAVA_HOME environment variable is not defined correctly` when running `service.bat` against the Java 9 JDK.

This is because Java 9 has done away with the `jre` folder, but the `service.bat` batch file expects this folder to exist.

See [this StackOverflow answer](https://stackoverflow.com/a/46388190) for more details on the new folder structure in Java 9.

The workaround is to install the Java 8 JDK alongside Java 9 and override the `JAVA_HOME` environment variable while running `service.bat`, or to use the `Windows Service Installer` exe to create the service.
:::
