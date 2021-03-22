---
title: Deploying to IBM WebSphere Liberty
description: Learn how to deploy to an IBM WebSphere Liberty web server with Octopus Deploy.
author: shawn.sesna@octopus.com
visibility: private
published: 2022-03-01-1400
metaImage: 
bannerImage: 
tags:
 - 
---

When evaluating software, it's easy to overlook an option because it doesn't list the specific stack you use.  However, just because it isn't listed, doesn't mean it isn't supported.  For Java-based applications, Octopus Deploy includes specific steps to deploy to Tomcat and Wildfly (JBOSS).  Though those are the two most popular options, there are more web server technologies available than just those two.  Web servers such as Payara and IBM WebSphere Liberty will automatically deploy an application once it's placed in a specific folder.  With this option available, writing a template for that specific server type isn't necessary.  In this post, I'll demonstrate how to deploy the PetClinic application to an IBM WebSphere Liberty web server.

## IBM WebSphere Liberty
[IBM WebSphere Liberty](https://www.ibm.com/cloud/websphere-liberty) is a web server technology for Java-based applications developed by IBM.  This server has both a [paid](https://www.ibm.com/cloud/websphere-liberty/pricing) and an [open source](https://openliberty.io/) variants.  Both variants have the same automatic application deployment option available when the application is placed within a specific folder for the server.