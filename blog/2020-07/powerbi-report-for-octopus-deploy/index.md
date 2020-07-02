---
title: How to build a PowerBI report for Octopus Deploy
description: Learn how to build PowerBI reports for Octopus Deploy that will show you deployment and runbook history in Octopus Deploy.
author: Jeff@ReviewMyDB.com
visibility: private
published: 2020-07-15
metaImage: octopus-power-bi-report.png
bannerImage: octopus-power-bi-report.png
tags:
  - DevOps
  - Reporting
---

![How to build a PowerBI report for Octopus Deploy](octopus-power-bi-report.png)

In this post, I share the analysis of the data I pulled from my Octopus Deploy database using PowerBI Desktop. I was able to highlight how much time and money automation has saved my company, and spot potential development issues that slow down deployments, such as how many times releases have been skipped due to bugs not being found in lower-level environments. At the end of this post, I share the templates I created so you don't have to start from scratch.

Hopefully, these reports will get you pointed in the right direction to help you highlight how much automation can save in time and money, and even justify the need for automation and testing resources.

This is a preview of what I put together. In this post, we’ll go over how you can create a connection and pull data from your instance of Octopus Deploy. The PowerBI templates I've included cover both your cloud and local instances to help you get started as quickly as possible.

![PowerBI Report For Octopus Deploy](headlinerimage.png "width=500")

This post is broken up into the following sections:

1. [Prerequisites](#prerequisites)
2. [Deployment history charts](#deployment-history-charts)
3. [Runbooks history charts](#runbooks-history-charts)
4. [Reports](#reports)
5. [ROI](#roi)
6. [Conclusion](#conclusion)

## Prerequisites

There are a couple of prerequisites you need to have in place:

- [Install PowerBI Desktop](https://blog.reviewmydb.com/2020/06/how-to-install-powerbi-desktop.html).
- [Connect to a local instance of Octopus Deploy](https://blog.reviewmydb.com/2020/06/how-to-connect-to-octopus-deploy-local.html).
- [Find your Octopus Cloud API key and connect to your cloud instance of Octopus Deploy](https://blog.reviewmydb.com/2020/06/how-to-connect-to-your-octopus-deploy.html).

## Deployment history charts {#deployment-history-charts}

Deployment history yields some very interesting patterns and results. From raw deployment counts with different groupings (project, environment etc) deployment execution time to success and failure rates.

**Count of Deployments By Date and Project**: This is the first chart I put together, and I was kind of shocked to see how many deployments we had done each year over the last five years. This report breaks down each year and stacks each project. In this chart, you can drill down to see the count of deployments by quarters, month, and day. You can drill down by clicking the _double down arrow_ and then click on the _up arrow_ to go back:

![](report01.png "width=500")

![](report01a.png "width=500")

**Count of Deployments By Environment and Project**: The chart below shows how many deployments we have done in each environment split by project:

![](report02.png "width=500")

**Total Deployments**: This is a count of all records in the Deployment History table, which is all of the deployments.

**Succeeded Deployments**: This is the count of all deployments, filtered by `TaskState` `Succeeded`.

**Failed Deployments**: This is the count of all deployments, filtered by `TaskState` `Failed`.

![](report03.png "width=500")

**Total Runtime Hours**: This is a measure that tells us the total time of all the deployments in hours. Duration is stored in seconds so we divide it to arrive at the total number of hours.

**Total Runtime Minutes**: This is a measure that tells us the total time of all the deployments in minutes. Duration is stored in seconds so again, we divide it to arrive at the total number of minutes.

**Total Runtime Seconds**: This is a sum of the total duration for all deployments.

![](report04.png "width=500")

**Count of Deployments by Status**: The chart below displays a summary and count of all deployments by task statuses:

![](report05.png "width=500")

**Average Runtime Seconds by Project**: The chart below shows how long each deployment takes for each project on average. This is for all environments:

![](report06.png "width=500")

**Total Deployments by Environment**: The chart below shows how many total deployments have occurred in each environment:

![](report07.png "width=500")

**Percentage of Issues by Project and Environment Based on Number of Deployments**: With the chart below, I wanted to show the ratio of deployments, or redeployments, we have done to each environment by project. This may not be an exact representation, but if the percentage of deployments to the lower environments (Dev and QA) are higher, this could be because issues were discovered and the deployments were fixed and then re-deployed. If we had an even percentage of deployments in each environment, it could mean that no bugs/issues were found and the deployment went through to Production.

If you notice in this chart that the _Store Website_ in both QA and Prod almost have the same percentage of deployments. To me, this means that instead of testing and finding issues in QA, the bugs are found in Production. This, to me, means that there is a code quality issue that needs to be addressed.

Compare the _Store Website_ to the _Website_, and you can see that the percentage difference is much higher in Dev and then QA compared to Production. I think this means the developers are finding bugs in Dev, and then the ones they miss are caught in QA. Over the past 5 years, we have released 16 major stable versions of the website.

While this may not be 100% accurate, you could pull in some data and create a relationship from your ticket system to make this more precise with supporting data.

![](report08.png "width=500")

**Percentage of Planned vs. Actual**: The chart below compares the releases from Development and Production. The goal was to see how many project releases were meant to go to Production, but due to either, code issues or bugs they had to be re-released before making it to Production:

![](report09.png "width=500")

**Applications with most Failed Deployments**: The chart below shows which project/application has had the most failed deployments. This could be for any number of reasons, such as misconfigured variables, changed servers, third-party component changes or upgrades, servers being down, failed scripts, etc. If there is a high number, this may be something to look into:

![](report11.png "width=500")

**Applications with the Most Deployments**: The chart below shows which project/application has had the most successful deployments.

![](report13.png "width=500")

**Average Days from Dev to Prod by Project**: The chart below shows the average duration in days from the time a release has been deployed to Dev and then the same release deployed to Production:

![](report15.png "width=500")

## Runbooks history charts {#runbooks-history-charts}

Runbooks were introduced in late 2019 and I've recently started using them for some basic operations tasks. These charts summarise runbook execution by date, execution time and success vs failures.

:::warning
These reports are currently only available on an on-premises instance of Octopus Deploy because there is no `RunbookHistory` API call on the cloud instance.
:::

**Frequency of Runbooks by Date**: The chart below shows the frequency of executions by date. Since we have only recently started using runbooks, I have drilled down to the day to see how many we have run this month:

![](report18.png "width=500")

**Runbook Executions**: This is a count of all records in the runbooks history table, which is all of the executions of all runbooks.

**Succeeded Runs**: This is the count of all runs, filtered by `TaskState` `Succeeded`.

**Failed Runs**: This is the count of all runs, filtered by `TaskState` `Failed`.

![](report19.png "width=500")

**Total Runtime Hours**: This is a measure that tells us the total time of all the runbook execution in hours. Duration is stored in seconds so we divide it to arrive at the total number of hours.

**Total Runtime Minutes**: This is a measure that tells us the total time of all the runbook execution in minutes. Duration is stored in seconds so we divide it to arrive at the total number of minutes.

**Total Runtime Seconds**: This is a sum of the entire duration for all runbook executions.

![](report20.png "width=500")

## Reports

It might seem like a lot of work to create these reports, and it is. But I have a treat for you! I have copied my PowerBI reports and created a template for both the SQL Server and the Cloud instances of Octopus Deploy. Below are links to each template and links to my blog, which covers setting up the connection string, API URL, and API key so that you can access your data.

- [SQL Server template: OctopusDeploySQLServer.pbt](octopus-deploy-sqlserver-report.pbit) ([Instructions](https://blog.reviewmydb.com/2020/06/how-to-configure-sql-server-powerbi.html)).
- [The Cloud template: OctopusDeployCloud.pbt](octopus-deploy-reporting-from-cloud-app.pbit) ([Instructions](https://blog.reviewmydb.com/2020/06/how-to-configure-powerbi-template-for.html)).

Now that we have our data, we can view all of the charts we created. I have organized the charts into several report _pages_.

The report contains four pages with a ton of useful information. 

1. Deployments
2. Issues
3. Durations
4. Runbooks

This is a preview of the frist page with a sample of the data from my local instance of Octopus Deploy for the last five years. The other pages contain the data that I summarised above as well as additional useful charts.

![Local instance: Deployments page](dashboard04.png "width=500")
*Local instance: Deployments page*

## ROI

So how much has Octopus Deploy saved my company? Let’s look at the data we have collected.

Over the past five years, Octopus Deploy has executed 3,572 deployments. Let’s say that on a typical manual deployment, each deployment takes one hour to deploy to multiple servers. In my case, we have at least one website, one API, and two databases. Our other environments have more web servers, which are in web farms, and more APIs.

At a bare minimum, that’s 3,572 hours, and we know for sure it would have taken more time for the larger environments. Manual execution of 3,572 hours, but the Octopus Deploy engines automated execution time tool only 53.5 hours.

If we had a person handling the deployments, this would take approximately one-third of their time each month just to perform deployments at about sixty hours out of a typical 168 hour work month.

If our hourly rate was $80 an hour, it would have cost us over $57,000 each year for five years for a total of $285,760.

Now, if I could convince my company to give me a small bonus out of the $267,360.00 I saved them, that would be awesome.

## Conclusion

Now that you can see your data, I hope that you find some of these reports and charts useful, and perhaps you will even think up of new reports and charts to add to this. If you do, comment and share your ideas, and perhaps I can incorporate them into my next version.
