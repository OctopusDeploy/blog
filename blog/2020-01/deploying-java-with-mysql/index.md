---
title: "Deploying Java with MySql backend through Octopus Deploy"
description: This post demonstrates how to deploy a Java application that uses a MySql backend using Octopus Deploy.
author: shawn.sesna@octopus.com
visibility: private
bannerImage: 
metaImage: 
published: 2020-01-13
tags:
 - DevOps
---

Octopus Deploy is well known for its ability to easily automate the deployment of .NET applications.  Though those were our roots, we've expanded our support to include things such as Java, Docker, and Kubernetes.  In this post, I walk through how to build a deploy a Java-based web application that uses a MySql backend database, then automate the updates to the web application and the database code using Octopus Deploy.

## Setting up the build server
When people think of TFS/Azure DevOps, they immediatly think .NET/.NET core, not Java.  However, the Microsoft build server comes with both Maven and ANT build tasks built-in to their task library!  Wait, that seems ... too easy.  You'd be right to be suspicious, though the tasks exist, they don't actually work without a little configuration :)  Luckily for us, it's all rather straight forward.

### Java on the build agent
To build Java, you need the Java Development Kit (JDK) on your build agent, which can be downloaded [here](https://www.oracle.com/technetwork/java/javase/downloads/index.html).  If you're a Windows guy like me, you'd think that running the installer on Windows would do everything necessary to make Java work.  Unfortunately, you'd be incorrect.  There are two additional steps necessary to make Java functional (at least on Windows):

- Creating the JAVA_HOME Environment Variable and setting it to the root of your Java installation (ie c:\Program Files\Java)
- Adding the \bin folder to the Path Environment Variable (ie c:\Program Files\Java\JavaVersion\bin)

### Maven on the build agent
The next thing we need to is install Maven on our build agent.  Maven doesn't have an installer and is merely a .zip file that needs to be extracted and placed on the build agent.  Similar to Java, we need to do a couple of things with Environment Variables:

- Create MAVEN_HOME and point it to where we extracted Maven to
- Adding the \bin folder to the Path Environment Variable

### Add the Maven capability
If you creating a new build agent, this step may not be necessary as part of the agent installation scans the machine for capabilities and will automatically add Maven if it's found.  If you're using an existing agent and adding Maven, you will need to go into Azure DevOps (ADO) and add the capability to the agent

Navigate to the Agent Pools section of ADO.  Select the agent you want to modify and click on Capabilities

![](ado-agent-pools.png)

Click on Add capability button and add the following

- JAVA_HOME 
- maven
- MAVEN_HOME

![](ado-add-capability.png)

With that work complete, our ADO instance should now be able to build Maven projects!

:::hint
The steps for configuring ADO to build ANT projects are nearly identical to these.  Replace the Maven with ANT for the build agent and capablity sections
:::

## The sample application
Ok, I cheated, I didn't write the sample application myself and instead relied on something [someone else](https://github.com/spring-petclinic/spring-framework-petclinic) had already built :)  Not being a Java guy, I needed something that both worked and used MySql, this repo ticked all the boxes and I was able to get it running in pretty short order!  (Okay, so there was a bit of a learning curve, but that was the fun part!).  This repo also has the added benefit of supporting PostGresSQL so there could be another blog post using this repo showing that :)

### Tweaking the POM