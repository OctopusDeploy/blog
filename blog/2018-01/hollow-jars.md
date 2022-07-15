---
title: An Introduction to Hollow JARs
description: Learn what Hollow JARs are and how you can create them from you existing WAR files.
author: matthew.casperson@octopus.com
visibility: public
published: 2017-11-07
metaImage: java-octopus-meta.png
bannerImage: java-octopus.png
tags:
 - DevOps
---

I have written in the past about [the difference between an application server and an UberJAR](https://octopus.com/blog/application-server-vs-uberjar).  The short version of that story is that an application server is an environment that hosts multiple JavaEE applications side by side, while an UberJAR is a self contained executable JAR file that launches and hosts a single application.

There is another style of JAR file that sits in-between these two styles called a Hollow JAR.

## What are Hollow JARs?

A Hollow JAR is a single executable JAR file that, like an UberJAR, contains the code required to launch a single application. Unlike an UberJAR though, a Hollow JAR does not contain the application code.

A typical Hollow JAR deployment then will consist of 2 files: the Hollow JAR itself, and the WAR file that holds the application code. The Hollow JAR is then executed referencing the WAR file, and from that point on the two files run much as an UberJAR would.

At first blush it might seem unproductive to have the two files that make up a Hollow JAR deployment instead of the single file that makes up the UberJAR, but there are some benefits.

The main benefit comes from the fact that the JAR component of the Hollow JAR deployment pair won’t change all that frequently. While you could expect to deploy new versions of the WAR half of the deployment multiple times per day, the JAR half will remain static for weeks or months. This is particularly useful when building up layered container images, as only the modified WAR file needs to be added as a container image layer.

Likewise you may also reduce build times with technologies like AWS autoscaling groups. Because the JAR file doesn’t change often, this can be baked into an AMI, while the WAR file can be downloaded as an EC2 instance is deployed through the scripting placed into an EC2 [user data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html) field.

## Building a Hollow JAR

To see Hollow JARs in action, let's take a look at how you can create one with [WildFly Swarm](http://wildfly-swarm.io/).

For this demo we will be building the pair of Hollow JAR deployment files required to run a copy of the [Ticket Monster](https://github.com/jboss-developer/ticket-monster) demo application. Ticket Monster is a sample application created to demonstrate a range of JavaEE technologies, and is designed to build a WAR file to run on a traditional application server.

To build the JAR half of the Hollow JAR, we will make use of [SwarmTool](https://wildfly-swarm.gitbooks.io/wildfly-swarm-users-guide/content/getting-started/tooling/swarmtool.html). Unlike WildFly Swarm, which usually requires special configuration in the Maven project to build an UberJAR or Hollow JAR, SwarmTool works by inspecting an existing WAR file and building a Hollow JAR to accommodate it. It is a neat way of migrating existing applications to the Swarm platform, without modifying the existing build process.

First, clone the Ticket Monster source code from https://github.com/jboss-developer/ticket-monster. The code we are interested in is under the `demo` subfolder.

There are two changes we need to make to the `pom.xml` file under the `demo` subfolder to accommodate Java 9 and SwarmTool.

First, we need to add a dependency on `javax.xml.bind:jaxb-api`. This is because the `java.xml` package is no longer part of [Java 9](https://stackoverflow.com/a/43574427/157605). If you try to compile the application under Java 9 without this additional dependency, you will receive the error:

```
java.lang.NoClassDefFoundError: javax/xml/bind/JAXBException
```

The following XML adds the required dependency:

```xml
<dependencies>
  <dependency>
        <groupId>javax.xml.bind</groupId>
        <artifactId>jaxb-api</artifactId>
        <version>2.3.0</version>
    </dependency>
</dependencies>
```

The second change is to embed the Jackson libraries used by Ticket Monster into the WAR file. In the original source code, the Jackson library has a scope of `provided`, which means that we expect the application server (or Hollow JAR in our case) to provide the library.

```xml
<dependency>
    <groupId>org.jboss.resteasy</groupId>
    <artifactId>resteasy-jackson-provider</artifactId>
    <scope>provided</scope>
</dependency>
```

However, the version of Swarm we will be using has a different version of the Jackson library to the one used by the Ticket Monster application. This mismatch means that the `@JsonIgnoreProperties` annotation used by Ticket Monster is not recognised by the version of the Jackson library provided by Swarm, resulting in some serialization errors.

Fortunately all that is required is to use the default scope, which will embed the correct version of the Jackson library into the WAR file. Dependencies embedded in the WAR file take precedence, and so the application will function as expected.

```xml
<dependency>
  <groupId>org.jboss.resteasy</groupId>
  <artifactId>resteasy-jackson-provider</artifactId>
</dependency>
```

We can now build the Ticket Monster application like any other WAR project. The following command will build the WAR file.

```
mvn package
```

Now we need to use SwarmTool to build the Hollow JAR. Download the SwarmTool JAR file locally.

```
wget https://repo1.maven.org/maven2/org/wildfly/swarm/swarmtool/2017.12.1/swarmtool-2017.12.1-standalone.jar
```

Then build a Hollow JAR.

```
java -jar swarmtool-2017.12.1-standalone.jar -d com.h2database:h2:1.4.196 --hollow target/ticket-monster.war
```

The `-d com.h2database:h2:1.4.196` arguments instruct SwarmTool to add the H2 in memory database dependencies to the Hollow JAR. SwarmTool can detect most of the dependencies required to boot the WAR file by scanning the classes referenced by the application code. However it can not detect dependencies like database drivers, so we need to manually tell SwarmTool to include this dependency.

The `--hollow` argument instructs SwarmTool to build a Hollow JAR that does not embed the WAR file. If we left this argument off, the WAR file would be embedded in the resulting JAR file, creating an UberJAR instead of a Hollow JAR.

At this point we have the two files that make up our Hollow JAR deployment. The WAR file at `target/ticket-monster.war` contains our application, while the `ticket-monster-swarm.jar` file is our Hollow JAR.

## Executing a Hollow JAR

To run the application, use the following command.

```
java -jar ticket-monster-swarm.jar target/ticket-monster.war
```

You can then open http://localhost:8080 to view the application.

![Ticket Monster](ticket-monster.png "width=500")

## Conclusion

Hollow JARs are a neat solution that provide a lot of flexibility in deployment strategies while retaining the convenience of an UberJAR. You can find more information on the different strategies from the blog post [The Skinny on Fat, Thin, Hollow, and Uber](https://developers.redhat.com/blog/2017/08/24/the-skinny-on-fat-thin-hollow-and-uber/).

If you are interested in automating the deployment of your Java applications, [download a trial copy of Octopus Deploy](https://octopus.com/downloads), and take a look at [our documentation](https://octopus.com/docs/deployments/java/deploying-java-applications).
