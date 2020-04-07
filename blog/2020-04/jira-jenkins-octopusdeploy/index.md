---
title: CI/CD feedback process
description: DevOps Nirvana - Integrating Jira, Jenkins, and Octopus Deploy
author: shawn.sesna@octopus.com
visibility: private
published: 2022-04-13
metaImage: 
bannerImage: 
tags:
 - DevOps
---

DevOps adoption has dramatically increased in recent years as people recognize the benefits it offers.  Integration of Continuous Integration (CI) solutions with Continuous Delivery (CD) solutions are now quite commonplace, though few provide any input to the continuous feedback loop of DevOps.  In this post, I will show you how to integrate Jenkins, Octopus Deploy, and Jira to provide a solution that will hook into the continuous feedback loop.

## Jenkins Octopus plugin
An Octopus Deploy plugin for Jenkins has existed for a number of years, however, this plugin was developed by the Jenkins community and not by Octopus Deploy.  In 2019, Octopus Deploy officially took over development and maintenance of the Jenkins plugin, implementing the same features that are avialable on other build platforms.

:::hint
Prior to Octopus Deploy taking ownership of the plugin, there was a security issue where passwords were stored in plain text.  This is no longer an issue and the warning that Jenkins would display has since been removed.
:::

### Installation of the Octopus Deploy plugin
Installation of the Octopus Deploy plugin is no different than installing any other plugin for Jenkins.  From the landing page in Jenkins, click Manage Jenkins

![](jenkins-managejenkins.png)

From here, click on Manage Plugins

![](jenkins-manage-plugins.png)

Click on the `Available` tab, then filter by Octo.  Tick the box next to Octopus Deploy and then choose either `Install without restart` or `Download now and install after restart`.

![](jenkins-manage-add-plugin.png)

Once the plugin has been installed, you will have access to the following build tasks
- Octopus Deploy: Package application
- Octopus Deploy: Push packages
- Octopus Deploy: Push build information

Along with those build tasks, you will also have the following Post Build tasks
- Octopus Deploy: Create Release
- Octopus Deploy: Deploy Release

:::hint
The Jenkins plugin differs from Azure DevOps, TeamCity, and Bamboo in that Create Release and Deploy Release are only available as Post build actions.  Jenkins only allows one of each type of Post build action type meaning you cannot have more than one Create Release action per build definition.  
:::

### Example build
For this post, I'm building the PetClinic application which is a Java application using MySQL as a backend.  To start, we'll select a New Item from the Jenkins menu

![](jenkins-new-item.png)

Give your project a name and slect `Maven project`

![](jenkins-maven-project.png)

Once you click `OK` you will be brought to the configuration screen for your build definition.

:::hint
I've configured my build to create a unique version number based on some parameters.  This version number will be stamped on the artifacts of the build that are later pushed to Octopus Deploy.  I've installed a couple of Jenkins plugins to make this work
- Build Name and Description Setter
- Date Parameter Plugin
:::

Under the `General Tab`, check the box `This project is parameterized`.

![](jenkins-build-general.png)

The required parameters of our build are (all `String` parameters):
- DatabaseName: #{Project.MySql.Database.Name}
- DatabaseServerName: #{Project.MySql.Database.Server.Name}
- DatabaseUserName: #{Project.MySql.Database.User.Name}
- DatabaseUserPassword: #{Project.MySql.Database.User.Password}

Optional parameters are (these are used to construct the version number, ie: 1.0.2098.101603):
- Major (String): 1
- Minor (String): 0
- Year (Date):
  - Date Format: yy
  - Default Value: LocalDate.now();
- DayOfYear (Date):
  - Date Format: D
  - Default Value: LocalDate.now();
- Time (Date):
  - Date Format: HHmmss
  - LocalDate.now();

![](jenkins-build-parameters.png)

With our parameters defined, let's hook the build into source control.  I'll be using the PetClinic public Bitbucket repo for this build:
https://twerthi@bitbucket.org/octopussamples/petclinic.git

![](jenkins-build-source-control.png)

Under the `Build Environment` tab, check the box `Set Build Name`.  This feature allows us to configure the build name to be the version number that we've configured with our parmaters (if you used the optional ones).  Set the `Build Name` to `${MAJOR}.${MINOR}.${YEAR}${DAYOFYEAR}.${TIME}



## Jira Integration
Jira is a popular software package used to track issues within software projects.  Jira is able to keep track of logged bugs, tasks, and epic items that are associated with your project.