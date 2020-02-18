---
title: "Beyond application deployment"
description: How to deploy SQL Server Reporting Services reports
author: shawn.sesna@octopus.com
visibility: private
bannerImage:
metaImage:
published: 2021-01-15
tags:
 - DevOps
---

Collecting and storing data is often the main functions of web applications.  The collected data needs to be analyzed and shown in graphical format to help make decisions.  This usually takes the form of a report.  In this part of my series, I will demonstrate how to include SQL Server Reporting Services (SSRS) reports using Octopus Deploy.

## Build the project
As of Visual Studio 2017, MSBuild can build .rtpproj files.  Projects that are created in older versions of Visual Studio will need to configure the build agents similar to my [SSIS post](/blog/2020-02/deploying-ssis/inde.md) in order to build the project.

### Add the build task
To build an SSRS project, simply add any MSBuild type task to your build definition.  For my build definition, I chose a Visual Studio build task, which invokes MSBuild:

![](ado-build-visual-studio-build.png)

### Package the artifacts
When the SSRS project is built, it places the built reports into the `bin` folder.  In this case, they are in `Reports/Reports/bin/release`  We to package these reports into either a .zip or a .nupkg package for deployment.  For this post, I choose the .nupkg format:

![](ado-pack-task.png)

### Push the artifacts
Now that we have the artifacts packaged, we can push them to our Octopus Deploy server:

![](ado-push-task.png)

And that’s all we need for our build definition.

## Octopus Deploy
Now that we have our artifacts in Octopus Deploy, we can create our project for deployment.

### Create the project
Click on **Projects**, then **ADD PROJECT**:

![](octopus-create-project.png)

### Deploy the package
The first step in our process is to deploy the package to the SSRS server. Click **ADD STEP**:

![](octopus-project-add-step.png)

Add a Deploy a Package step and select the package we pushed to the Octopus Server:

![](octopus-project-add-deploy-package.png)

After the step has been added, fill in the text boxes:

![](octopus-project-deploy-package1.png)
![](octopus-project-deploy-package2.png)

### Add the SSRS step

To deploy SSRS reports, I add a Community Step template.  Filter steps by SSRS and select Deploy SSRS Reports from a package step:

![](octopus-project-ssrs-step.png)

This step needs to execute on the same target that the package was deployed, for the Execution Location, choose Deployment Target and fill in the same role you chose for the Deploy a package step:

![](octopus-project-ssrs-step1.png)

Fill in the values for the step.

- **SSRS Package Step**: This is the Deploy a Package step we created previously.
- **URL of SSRS Server service**:  This is the URL of the Report Service web service.  For example, http://servername/ReportServer/reportservice2010.asmx?wsdl.
- **Report Execution Url**:  This is the URL of the Report Execution web service.  For example, http://servername/ReportServer/ReportExecution2005.asmx?wsdl.
- **Report Folder**: The Relative path to the folder where the reports will be deployed.  For example, `/MyFolder`.
- **Report data source folder**: The relative path to the folder where the datasources will be deployed.  For example `/MyFolder/MyDataSources`.
- **Overwrite Datasources**: Check this if you want to overwrite the datasources when deploying.

:::hint
To specify the datasource details, you need to create Octopus Deploy variables that match the datasource to overwrite.  The following properties can be overridden:

- ConnectionString
- Username
- Password

For example, if the name of the datasource is `MyDatasource`, the variables would be:

- MyDatasource.ConnectionString
- MyDatasource.Username
- MyDatasource.Password

If the username and password are for a domain account, you need to create an additional variable called `MyDatasource.WindowsCredentials` with a value of `True`.
:::

- **Backup Location (optional)**:  The location on the Deployment Target to back up reports before overwriting them.
- **DataSet Folder (optional)**:  The relative path to the folder where Shared DataSets will be deployed.  For example `/MyFolder/MyDatasets`.
- **Report Parts Folder (optional)**: The relative path to the folder where Report Parts will be deployed.  For example `/MyFolder/MyReportParts`.
- **Service Domain (optional)**:  Domain name of the account to use when deploying.
- **Service Username (optional)**:  Username of the account to use when deploying.
- **Service Password (optional)**:  Password for the user to use when deploying.
- **Clear the report folder**: Check this box if you want to delete reports from the target folder prior to deployment.

![](octopus-project-ssrs-step-complete.png)

That’s all for the steps.  We’re now ready to deploy.

## Deploy the reports
With our process complete, let’s create a release. Click **CREATE RELEASE**:

![](octopus-project-create-release.png)

After you click **SAVE**, choose the deployment environment:

![](octopus-project-create-deployment1.png)

Click **DEPLOY**:

![](octopus-project-create-deployment2.png)

After the deployment is complete, it should look like this:

![](octopus-project-deploy-complete.png)

## Conclusion
In this post, I demonstrated how easy it is to deploy SSRS reports using Octopus Deploy. Happy Deployments!
