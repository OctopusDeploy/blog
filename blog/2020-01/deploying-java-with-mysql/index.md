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
If you are creating a new build agent, this step may not be necessary as part of the agent installation scans the machine for capabilities and will automatically add Maven if it's found.  If you're using an existing agent, you will need to go into Azure DevOps (ADO) and add the capability to the agent manually.

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
Ok, I cheated.  I didn't write the sample application myself. Instead, I found a [great sample](https://github.com/spring-petclinic/spring-framework-petclinic) already built :)  Not being a Java guy, I needed something that both worked and used MySql, this repo ticked all the boxes and I was able to get it running in pretty short order!  (Okay, so there was a bit of a learning curve, but that was the fun part!).

### Tweaking the POM
There were some tweaks I needed to make to the POM.XML (Maven Project Object Model) file to make it work for this post:

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
The author(s) of this repo did a fantastic job of making this application support multiple database backends; HyperSQL, MySQL, and PostgreSQL.  The default is set to the HyperSQL profile (HSQLDB).  To change it to MySQL was a simple matter of moving the `<activation>` XML node from the HSQLDB profile to the MySQL profile.  To do this, find the `<profiles>` XML node in the POM.XML file.  Locate the `<profile>` node that has a child node of `<id>HSQLDB</id>`.  Directly underneath the `<id>` node is an `<activation>` node.  Move `<activation>` node to the MySQL node.  The resulting MySQL node should look like this:

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
When the Maven project is compiled, the properties of the active database profile get copied into the `/WEB-INF/classes/spring/datasource-config.xml` file within the resulting .war archive.  The `datasource-config.xml` is the file that is used for the connection string to the database for this application.  In the above code sample, we've changed the connection string for the server and database to use the variables of `databaseServerName` and `databaseName`

```
<jdbc.url>jdbc:mysql://${databaseServerName}/${databaseName}?useUnicode=true</jdbc.url>
```

#### Alter the finalName attribute
The `finalName` attribute of the POM.XML is what the name of the .war archive will be when the project is packaged.  The default finalName is:

```
<finalName>petclinic</finalName>
```
When built, this results in a filename of petclinic.war.  Octopus Deploy uses [Semantic Versioning](https://semver.org/) which has a version number embedded in the file name.  The `version` attribute of the POM.XML file will work perfectly for this as we've already made it dynamic.  Update your `<finalName>` attribute to:

```
<finalName>petclinic.web.${project.version}</finalName>
```

I added the `.web.` part to more easily identify which component this is within Octopus Deploy :)

#### Update cssDestinationFolder attribute
As I was going through the exercise of getting this to work on my local system, I came across a nasty surprise when I changed the finalName attribute of the POM, none of the css made it into to final product.  After a bit of troubleshooting, I found that the action that copies the css into the application uses the finalName value for the path to copy to.  To fix it, I needed to update the `<cssDestinationFolder>` attribute from:

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
There was one last thing that I learned with this example application, it ran the included database scripts whenever it was deployed.  After a bit of digging, I found that I could comment out some XML in the datasource-config.xml file that would stop it from doing that.  I still need the database scripts, I just didn't want them to execute every time the application was deployed.  More on this later.

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
## Adding a Flyway project
[Flyway](https://flywaydb.org) is a free, migrations-based database deployment tool.  In a nutshell, it's a command-line utility that you include in your project that uses a specific folder structure to execute SQL scripts in a specified order.  The download of Flyway is essentially the project that you will add to your project source control.

### Adding the .sql scripts to Flyway
Within the Java application source, copy the .sql files located in src/main/resources/db/mysql to the /sql folder of your Flyway project.  Once the files are there, rename them to conform to [Flyway works](https://flywaydb.org/getstarted/how).  This is what mine looked like:

- V1__initDb.sql
- V1_1__populateDb.sql

And that's it!  Pretty simple, huh?  :)

## Creating the build definition
Now that we've done the prequisite work of installing Maven on the build agent, and tweaked a couple of files, and adding Flyway, we're in a positon to create our build definition!

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

If you're not familiar with the #{} syntax used for the variable value, it's Octostache which is used for variable replacement in Octopus Deploy.

![](ado-project-variables.png)

You'll note that there are two additional variables defined; MajorVersion and MinorVersion.  I use these variables to create my Build Number format within ADO.  To set this, click on the Options tab and fill in the Build number format, I use $(MajorVersion).$(MinorVersion).$(Year:yy)$(DayOfYear).$(Date:Hmmss)

![](ado-build-number-format.png)

This is the value that the ADO reference of $(Build.BuildNumber) uses which we are feeding into the project.VersionNumber variable we are passing into the Maven build task.

We're done wi the Maven task!  .war is a supported file type for the built-in package repository in Octopus Deploy, so there's no need to package up the Java application.

### Packaging the Flyway project
As previously mentioned, Flyway is a command-line utility, so there's no building of the project.  The only thing we need to do is package it up so that Octopus Deploy can do something with it.  For this, we can use any task that creates a ZIP or NuGet package.  This demonstration will use the Octopus Deploy plugin.  

- Package ID: petclinic.flyway
- Package Format: NuPkg
- Package Version: $(Build.BuildNumber)
- Source Path: $(Build.SourcesDirectory)\flyway
- Output Path: $(build.stagingdirectory)

![](ado-build-add-flyway-package.png)


### Pusing to Octopus Deploy
The last part of our build process is going to be pushing the packages that we created to our Octopus Deploy server.

- Space: Select the space you're using
- Package
 - `$(Build.SourcesDirectory)\target\*.war`
 - `$(build.artifactstagingdirectory)\*.nupkg`

 ![](ado-build-push-packages.png)

 That's it for your build definition!  (Phew!)

 ## Deploying with Octopus Deploy
 I know what you're thinking, "Shawn, is setting up the project in Octopus going to be as long as setting the build?"  No!  We've done the hard part already, setting up the deployment project is pretty easy :)  "Easy for you to say, you work for 'em!"  No, seriously, it's actually simple.

 ### Creating the Octopus Deploy project
 Once you get into the Octopus Deploy UI, click on the Projects tab, then click on ADD PROJECT

 ![](octopus-create-project.png)

 Give the Project a name then click SAVE.  Optionally, you can select which Project group to put it in and what Lifecycle to use.

![](octopus-project-name.png) 


#### Variables
After we click SAVE, it will take us directly into our brand new project!  Let's click on Variables and define some of those real quick, we'll need them to exist first before we start defining our process.

![](octopus-project-variables.png)

On the Variables screen, we are going to create the following variables:
- Project.MySql.Database.Name
- Project.MySql.Database.ServerName
- Project.MySql.Database.User.Name
- Project.MySql.Database.User.Password

Namespacing your variables is considered a Best Practice for Octopus Deploy, it helps users identify where the variable is coming from and what it's used for.

##### Project.MySql.Database.Name
This variable is one of the variables that we set up with our build process using the OctoStache syntax.  We'll cover how the variable is replaced in just a little bit.  For now, give this variable the value of your database name.  In my case, I used: petclinic

##### Project.MySql.Database.ServerName
It is often the case that the database server is different as you go from one environment to another.  This variable will use the value that is scoped to the environment being deployed to.

##### Project.MySql.Database.User.Name
As the name suggests, this is the User Name for the database connection.

##### Project.MySql.Database.User.Password
This is the password for the user account for the database connection.

![](octopus-project-variables-values.png)

#### Process
With our variables filled in, let's define our process!  Click on the Process button on the left-hand side.

![](octopus-project-process-tab.png)

##### Add Deploy Package Step
Once on the Process screen, click on the ADD STEP button

![](octopus-project-add-step.png)

Choose the Package category and the Deploy a Package step

![](octopus-project-add-deploy-package.png)

Fill in the properties of the step:
- Step Name: Deploy Flyway package
- On Targets Roles: PetClinic-Db (this is what I named the role)
- Package: petclinic.flyway

(Picture shown is after SAVE has been clicked, screen is smaller :P)

![](octopus-flyway-package-step.png)

###### Variable replacement
Optional - If you've placed any OctoStache variable placeholders in any of your .sql scripts (I did), you'll need to configure this step to perform the variable replacement.

Click on the COFIGURE FEATURES button

![](octopus-project-step-configure-features.png)

Select `Substitute Variables in Files`

![](octopus-project-step-variable-replacement.png)

Click OK, then scroll down and expand the Substitute Variables in Files section.  For `Target files`, enter the value sql/*.sql and click SAVE

![](octopus-project-step-variable-replacement-sql.png)

##### Add the Flyway Migrate step
The Flyway step is not a built-in step to Octopus Deploy.  This step was developed by the Octopus Deploy community and added to our Community Step Template Library.

Just like before, click on the ADD STEP button.  When the window comes up, type in flyway to filter the steps.  Immediatly, you'll notice all categories except Community Steps get grayed out.  Mouse-over the Flyway Migrate and click on Install and Add.

![](octopus-community-steps.png)

![](octopus-community-flyway.png)

You'll be prompted to Install and Add, click SAVE

![](octopus-community-install.png)

Fill in the required properties of the step template
- On Targets in Roles: PetClinic-Db (this is what I named the role)
- Flyway package step: Use the drop down to select the Deploy Flyway package step
- Target -url: jdbc:mysql://#{Project.MySql.Database.ServerName}/#{Project.MySql.Database.Name}?useUnicode=true
- Target -user: #{Project.MySql.Database.User.Name}
- Target -password: #{Project.MySql.Database.User.Password}
Since this is a sensative variable, we need to click on the Bind icon to set it to use a variable
![](octopus-project-step-bind.png)
![](octopus-project-step-password.png)

That's it for this step, click SAVE!

(At the time of this writing, the Deploy a Package step and the Flyway migrate step must be executed on a target.  However, a worker friendly version of Flyway Migrate is under review)

##### Deploying the website step
Octopus Deploy has built-in steps for the two most popular web servers for Java-based applications, Tomcat and JBOSS/Wildfly.  This demonstration is going to make use of the Wildfly step.

Like before, click ADD STEP, then filter by Wildfly.  Choose the `Deploy to Wildfly or EAP` step template

![](octopus-project-add-wildfly.png)

Fill in the details of the step template
- Step Name: Deploy Petclinic Web
- On Targets in Roles: Petclinic-Web
- Package ID: petclinic.web
- Management host or IP: #{Octopus.Machine.Hostname} (this is a system variable that will use the hostname of the machine being deployed to)
- Management user: [Your management user]
- Management password: [Password for management account]
- Deployment name: PetClinic.war

That's it!  You're done!
