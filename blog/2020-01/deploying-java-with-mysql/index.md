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

Octopus Deploy is well known for its ability to easily automate the deployment of .NET applications.  Though those are our roots, we've expanded our support to include things such as Java, Docker, and Kubernetes.  In this post, I walk through how to build a deploy a Java-based web application that uses a MySql backend database, then automate the updates to both the web application and the database code using Octopus Deploy.

## Setting up the build server
For this demonstration, I used Azure DevOps as my build server.  When people think of Azure DevOps, they immediatly think .NET/.NET core, not Java.  However, the Microsoft build server comes with both Maven and ANT build tasks built-in to their task library!  Wait, that seems ... too easy.  You'd be right to be suspicious, though the tasks exist, they don't actually work without a little configuration :)  Luckily for us, it's all rather straight forward.

### Java on the build agent
To build Java, you need the Java Development Kit (JDK) on your build agent, which can be downloaded [here](https://www.oracle.com/technetwork/java/javase/downloads/index.html).  If you're a Windows guy like me, you'd think that running the installer on Windows would do everything necessary to make Java work.  Unfortunately, you'd be incorrect.  There are two additional steps necessary to make Java functional (at least on Windows):

- Creating the JAVA_HOME Environment Variable and setting it to the root of your Java installation (ie c:\Program Files\Java)
- Adding the \bin folder to the Path Environment Variable (ie c:\Program Files\Java\JavaVersion\bin)

### Maven on the build agent
The next thing we need to is install Maven on our build agent.  Maven doesn't have an installer and is merely a .zip file that needs to be extracted and placed on the build agent.  Similar to Java, we need to do a couple of things with Environment Variables:

- Create MAVEN_HOME and point it to where we extracted Maven to
- Adding the \bin folder to the Path Environment Variable

### Add the Maven capability
If you are creating a new build agent, this step may not be necessary as part of the agent installation scans the machine for capabilities and will automatically add Maven if it's found.  If you're using an existing agent, you will need to go into Azure DevOps (ADO) and add the capability to the agent

Navigate to the Agent Pools section of ADO.  Select the agent you want to modify and click on Capabilities

![](ado-agent-pools.png)

Click on Add capability button and add the following

- JAVA_HOME 
- maven
- MAVEN_HOME

![](ado-add-capability.png)

With that work complete, our ADO instance can now be able to build Maven projects!

:::hint
The steps for configuring ADO to build ANT projects are nearly identical to these.  Replace the Maven with ANT for the build agent and capablity sections
:::

## The sample application
Ok, I cheated.  I didn't write the sample application myself. Instead, I found something [someone else](https://github.com/spring-petclinic/spring-framework-petclinic) had already built :)  Not being a Java guy, I needed something that both worked and used MySql, this repo ticked all the boxes and I was able to get it running in pretty short order!  (Okay, so there was a bit of a learning curve, but that was the fun part!).  This repo also has the added benefit of supporting PostgreSQL so there could be another blog post using this repo showing that ;)

### Tweaking the POM
There were some tweaks I needed to make to the POM.XML (Maven Project Object Model file) to make it work for this post:

- Make the `<version>` attribute dynamic using a variable
- Switch the active profile to MySql
- Alter the jdbc.url of the MySql profile to use variables
- Alter the finalName attribute
- Update cssDestinationFolder attribute

#### Making the version number dynamic
The version number in the original project was hardcoded, I wanted to make this a dynamic value based on the build number.  This is easily accomplished with the use of variables which could be pumped into the Maven build process.  Simply change

```
...
<version>5.2.1</version>
...
```
to
```
...
<version>${project.versionNumber}</version>
...
```
:::hint
The variable name `project.versionNumber` is merely the name that I chose, you can name it whatever you want
:::

#### Change the active database profile to MySql
The author(s) of this repo did a fantastic job of making this application support multiple database backends, HyperSQL, MySQL, and PostgreSQL.  The default is set to the HyperSQL profile (HSQLDB).  To change it to MySQL was a simple matter of moving the `<activation>` XML node from the HSQLDB profile to the MySQL profile.  To do this, find the `<profiles>` XML node in the POM.XML file.  Locate the `<profile>` node that has a child node of `<id>HSQLDB</id>`.  Directly underneath the `<id>` node is an `<activation>` node.  Move `<activation>` node to the MySQL node.  The resulting MySQL node should look like this:

```
        <profile>
            <id>MySQL</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>            
            <properties>
                <db.script>mysql</db.script>
                <jpa.database>MYSQL</jpa.database>
                <jdbc.driverClassName>com.mysql.jdbc.Driver</jdbc.driverClassName>
                <jdbc.url>jdbc:mysql://${databaseServerName}/${databaseName}?useUnicode=true</jdbc.url>
                <jdbc.username>root</jdbc.username>
                <jdbc.password></jdbc.password>
            </properties>
            <dependencies>
                <dependency>
                    <groupId>mysql</groupId>
                    <artifactId>mysql-connector-java</artifactId>
                    <version>${mysql-driver.version}</version>
                    <scope>runtime</scope>
                </dependency>
            </dependencies>
        </profile>
```

#### Alter the jdbc.url of the MySql profile to use variables
When the Maven project is compiled, the properties of the active database profile get copied into the `/WEB-INF/classes/spring/datasource-config.xml` file within the resulting .war file.  The `datasource-config.xml` is the file that is used for the connection string to the database for this application.  In the above code sample, we've changed the connection string for the server and database to use the variables of `databaseServerName` and `databaseName`

```
<jdbc.url>jdbc:mysql://${databaseServerName}/${databaseName}?useUnicode=true</jdbc.url>
```

#### Alter the finalName attribute
The `finalName` attribute of the POM.XML is what the name of the .war file will be when the project is packaged.  The default finalName is:

```
<finalName>petclinic</finalName>
```
When built, this results in a filename of petclinic.war.  Octopus Deploy uses [Semantic Versioning](https://semver.org/) which has a version number embedded in the file name.  The `version` attribute of the POM.XML file will work perfectly for this as we've already made it dynamic.  Update your `<finalName>` attribute to:

```
<finalName>petclinic.web.${project.version}</finalName>
```

I added the `.web.` part to more easily identify which component this is :)

#### Update cssDestinationFolder attribute
As I was going through the exercise of getting this to work on my local system, I came across a nasty surprise when I changed the finalName attribute of the POM, none of the css made it into to final product.  After a bit of troubleshooting, I found that the action that copies the css into the application uses the finalName.  To fix it, I needed to update the `<cssDestinationFolder>` attribute from:

```
<cssDestinationFolder>${project.build.directory}/petclinic/resources/css</cssDestinationFolder>
```
to
```
<cssDestinationFolder>${project.build.directory}/petclinic.web.${project.version}/resources/css</cssDestinationFolder>
```
:::hint
If you're application renders like
![](css-folder-incorrect.png)

your `<cssDestinationFolder>` is incorrect
:::

If your recall I mentioned the learning curve, there you have it ;)  There is one more piece we need to do, but it's not in the POM.

### Updating datasource-config.xml
There was one last thing that I learned with this example application, it ran the included database scripts whenever it was deployed.  After a bit of digging, I found that I could comment out some XML in the datasource-config.xml file that would stop it from doing it.  I still need the database scripts, I just didn't want them to execute every time the application was deployed.  More on this later.

Navigate to `/src/main/resources/spring/datasource-config.xml` and comment out the Database initiliazer section.  It should look like this:

```
    <!-- Database initializer. If any of the script fails, the initialization stops. -->
    <!-- As an alternative, for embedded databases see <jdbc:embedded-database/>. -->
    <!--
    <jdbc:initialize-database data-source="dataSource">
        <jdbc:script location="${jdbc.initLocation}"/>
        <jdbc:script location="${jdbc.dataLocation}"/>
    </jdbc:initialize-database>
    -->
```
With those files updated, we can proceed with build definition!

## Creating the build definition
Now that we've done the prequisite work of installing Maven on the build agent, and tweaked a couple of files, we're in a positon to create our build definition!

### Adding the Maven task
Create a new build definition, this demonstration is using the classic editor instead of the YAML approach.

![](ado-create-new-build.png)

Start with an empty job

![](ado-empty-job.png)

Add the Maven build task

![](ado-add-maven-task.png)

Fill in the task input fields

Display name: 
This value doesn't matter

Maven POM file: 
If you're pom.xml is not in the root folder, use the elipses (...) to locate it

Goal(s): 
clean package dependency:purge-local-repository

:::hint
the dependency:purge-local-repository goal may not be necessary, but I like to clean my sources folder when building
:::

Options:
-Dproject.versionNumber=$(Build.BuildNumber) -DdatabaseServerName=$(DatabaseServerName) -DdatabaseName=$(DatabaseName) -DskipTests=$(SkipTests)

![](ado-maven-fields.png)

Hop over to Variables and create the following
- DatabaseName: #{Project.MySql.Database.Name}
- DatabaseServerName: #{Project.MySql.Database.ServerName}
- SkipTests: true

If you're not familiar with the #{} syntax used for the variable value, that's Octostache which is used for variable replacement in Octopus Deploy