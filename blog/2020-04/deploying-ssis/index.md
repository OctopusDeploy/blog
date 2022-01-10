---
title: Deploying SQL Server Integration Services (SSIS) packages with Octopus
description: How to deploy SQL Server Integration Services packages with Octopus Deploy.
author: shawn.sesna@octopus.com
visibility: public
bannerImage: deploying-ssis.png
bannerImageAlt: Deploying SSIS with Octopus Deploy
metaImage: deploying-ssis.png
published: 2020-04-14
tags:
 - DevOps
 - Database Deployments
---

![Deploying SSIS with Octopus Deploy](deploying-ssis.png)

When you think of automating application deployments, what usually comes to mind is automating the deployment web code, containers, and or database.  In this series, I’ll demonstrate how to automate supporting components such as SQL Server Integration Services (SSIS) packages and SQL Server Reporting Services (SSRS) reports.

- **Deploying SQL Server Integration Services (SSIS) packages with Octopus**
- [Deploying SQL Server Reporting Services (SSRS) reports with Octopus](/blog/2020-04/deploying-ssrs/index.md)

---

Extract Transform and Load (ETL) processes are typically deployed by a database administrator (DBA) or an integrations specialist.  In lower-level environments, it’s not unusual to allow developers to deploy directly to the SSIS server, but when it comes to higher-level, such as production, they‘re usually done manually.  The DBAs sometimes have scripts they can run to deploy, but it’s usually not included in the same tooling as automated application deployments.  With Octopus Deploy, it’s possible to add your ETL deployments to your deployment stack.

## Building the SSIS package
To include our SSIS package in the deployment process, you first need to build the project to produce the artifact for deployment.  None of the popular build servers can build SSIS out of the box and will require some configuration.  The steps will be roughly the same for all build servers.

:::hint
Your SSIS project **must** be in the Project Deployment Model, the Package Deployment Model will not work with this solution.
:::

### Configuring the build agent
MSBuild does not know how to build SSIS project types.  In order to perform a build, you need to configure the build agent.

#### Visual Studio
To build SSIS projects, we will need to install Visual Studio on our build agent.  The community edition of Visual Studio should be enough.

:::warning
Consult the EULA of the community edition to make sure it is OK to use for you or your organization.
:::

#### SQL Server Data Tools
After we have Visual Studio on our build agent, we need to install an extension called SQL Server Data Tools (SSDT).  When that has been installed, our build agent will have the ability to build .dtsproj projects.

### Build SSIS task
Out-of-the-box, most if not all build servers do not have a task to build SSIS by default.  You need to install or configure custom tasks to perform the build.

#### Azure DevOps/Team Foundation Server
The Marketplace for Azure DevOps (ADO) or Team Foundation Server (TFS) has several tasks  the community has created.  If you’re using ADO/TFS, I suggest using this one:

![](build-ado-ssis-task.png)

#### TeamCity
TeamCity doesn’t have a plugin available to perform the build, but Pavel Hofman has a great [blog post](https://blog.codetitans.pl/post/howto-setup-ssis-project-to-build-via-teamcity/) on how to configure building SSIS projects on TeamCity.

#### Jenkins
Similar to TeamCity, Jenkins does not have any plugins available to perform a Visual Studio build.  However, performing similar steps to the TeamCity solution should yield the same results.

### Building the project
In this post, I use Azure DevOps to perform the build.

#### Build task
Fill in the build task details:

![](ado-ssis-task.png)

#### Packaging the artifact
After the SSIS project has been built, it will produce an .ispac file, which contains the necessary components for deployment.  .ispac isn’t a standard archive like .zip or .nupkg, so you need an additional step that packages it into a supported format.  For ADO, TeamCity, and Jenkins, Octopus Deploy has plugins or extensions that contains steps that do this:

![](ado-package-task.png)

#### Pushing the artifact to a repository
After the .ispac file has been packaged, you need to package it and send it to a repository, such as:

- Octopus Deploy built-in repository
- Artifactory
- Nexus
- ADO/TFS repository

![](ado-push-task.png)

## Octopus Deploy
Now that we have the package ready, we can create and configure our Octopus Deploy project.

### Create the project
To create our project, click on **Projects** and **ADD PROJECT**:

![](octopus-create-project.png)

#### Add SSIS deployment step
The only step templates for deploying SSIS packages are in the Community Step Library.  Click on **ADD STEP**:

![](octopus-project-add-step.png)

Filtering by SSIS will show the available SSIS step templates.  For this demonstration, I use `Deploy ispac SSIS project from Referenced Package`.  This template will allow us to deploy using a worker instead of installing an agent on the SSIS server.

![](octopus-project-step-ssis.png)

#### Fill in the step details
This step allows us to run the step on either a deployment target or a worker.  For this demonstration, I use a worker, so we don’t need to install a Tentacle on the SSIS server itself.  Expand the `Execution Location` section and select `Run once on a worker`.

![](octopus-project-step-worker.png)

Now fill in the parameters:

- **Database server name (\instance)**: The name of the SSIS server to connect to.  (i.e., SSISServer1 or SSISServer1\Instance1).
- **SQL Authentication Username (optional)**: The SQL account username. Leave this blank to use Integrated Authentication.
- **SQL Authentication Password (optional)**: The password to the SQL account. Leave this blank to use Integrated Authentication.
- **Enable SQL CLR**: The SSISDB feature of SQL Server requires that the SQL CLR be enabled.  If the feature isn’t already enabled, set this to true.
- **Catalog name**: The name of the catalog for SSISDB, it is recommended not to change this value.  This is only needed if the SSISDB feature has not already been installed.
- **Catalog Password**: The password for the SSISDB catalog.  This is only needed if the SSISDB feature has not already been installed.
- **Folder name**:  Name of the folder to place the SSIS project within the SSISDB catalogue.
- **Project name**: The name of the SSIS project.  This name must match exactly the name of the project within Visual Studio.
- **Use Environment**: Set to true if you want to use environment variables in SSISDB.
- **Environment name**: The name of the environment to use.
- **Reference project parameters to environment variables**: Set to true to link project variables to environment variables.
- **Reference package parameters to environment variables**: Set to true to link package variables to environment variables.
- **Use fully qualified variable names**: When true, the package variables names must be represented in `dtsx_name_without_extension.variable_name`.
- **Use Custom Filter for connection manager properties**: Custom filter should contain the regular expression for ignoring properties when setting will occur during the auto-mapping.
- **Custom Filter for connection manager properties**: Regular expression for filtering out the connection manager properties during the auto-mapping process. This string is used when `UseCustomFilter` is set to true.
- **Package ID**: ID of the package used for deployment.
- **Package Feed ID**: The ID of the feed where the package resides.

![](octopus-project-step-ssis1.png)

![](octopus-project-step-ssis2.png)

![](octopus-project-step-ssis3.png)

With the form filled out, we can now deploy our package.

### Deploy
Let’s create our release.  Click the **CREATE RELEASE** button:

![](octopus-project-create-release.png)

Click **SAVE**:

![](octopus-project-create-release-save.png)

Select the environment to deploy to:

![](octopus-project-create-release-deploy.png)

Then confirm the deployment:

![](octopus-project-create-release-deploy2.png)

Our package has been deployed:

![](octopus-project-deploy-complete.png)

#### Deployment log
If you’ve referenced project parameters to environment variables, you’ll note something like the following in the deployment log:

```
- Adding environment variable CM.WWI_Source_DB.ConnectionString
**- OctopusParameters collection is empty or CM.WWI_Source_DB.ConnectionString not in the collection -**
```
This message indicates that you’ve referenced a project parameter to an environment variable, but the variable `CM.WWI_Source_DB.ConnectionString` was not found in the Octopus Deploy project variables.  This means you can create an Octopus Deploy project variable with the same name to control the value that goes from environment to environment, just like when you deploy an application.

:::hint
Creating the variables by hand can sometimes be time consuming.  See my [importing variables from SSISDB](https://octopus.com/blog/get-variables-from-ssisdb) post to automate variable creation.
:::

#### Viewing the results
Let’s take a look at our SSISDB using SQL Server Management Studio (SSMS):

![](ssms-ssisdb-ssispackage.png)

Opening the environment, we see our variables have been created:

![](ssms-ssisdb-environmentvariables.png)


## Conclusion
In this post, I demonstrated how to deploy SSIS packages using Octopus Deploy.  Using this method, you can now include supporting application components using the same tooling.
