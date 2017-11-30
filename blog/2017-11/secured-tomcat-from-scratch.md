---
title: Deploying a Secured Web App to Tomcat with Octopus
description: Learn how to go from a fresh Tomcat download to a secured web app deployment using the new features in Octopus 4.1
author: matthew.casperson@octopus.com
visibility: private
metaImage: java-octopus-meta.png
bannerImage: java-octopus.png
tags:
 - Java
---

## Download Tomcat 9

For this demo we'll use Tomcat 9 on Windows 2016. You can download Tomcat 9 from [here](https://tomcat.apache.org/download-90.cgi). Make sure to grab the Windows 64 bit zip version, as this contains a number of extras, like the `tcnative-1.dll` file, which are quite handy to have.

Extract the ZIP file into `C:\apache-tomcat-9.0.1`. This directory is referred to as `CATALINA_HOME`.

## Download Octopus Deploy 4.1

Grab a copy of Octopus Deploy 4.1 from the [downloads page](https://octopus.com/downloads). Version 4.1 has includes a number of new steps and features for integrating with Maven repos and deploying certificates. You can find more information on installing Octopus from the [documentation](https://octopus.com/docs/installation).

:::hint
Octopus 4.1 is currently in beta, so if it is not available from the downloads page now, it will be soon. Watch this space!
:::

http://repo1.maven.org/maven2/com/github/gwtmaterialdesign/gwt-material-demo/2.0-rc5/
