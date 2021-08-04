---
title: Telemetry in Octopus Crows Nest
description: Learn how Octopus is using telemetry data to identify performance metrics for our customers
author: terence.wong@octopus.com
visibility: private # DO NOT CHANGE THIS!!!! This is not a public post!
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

As Octopus Deploy has grown, Octopus Deploy now covers a large software surface area. A given user may need to interface with many different touchpoints across Octopus Deploy. These may be viewing the dashboard, securely signing on through certificates or viewing the projects page. In order to serve our customers better, we want to be able to tell whether users accessing these touchpoints are receiving their content in a timely manner.

Telemetry is the process of collecting measurements or statistical data and forwarding it to IT systems in a remote location. Many software companies use it to gather data on how their customers use and experience their product.

Octopus Deploy has implemented a telemetry tool that tracks how quickly customers are served web requests. Telemetry is activated by default unless users choose to opt-out. A web request is sent when a customer wants to visit a page or poll an endpoint to get a response. Web requests over several Octopus Deploy versions were tracked. The following categories are sent:

- Web request duration (2020.4)
- Database operation duration (2020.4)
- This should not happen (2021.1)
- Background job duration (2021.2)

Only web requests with a 2xx status are used in the calculations, however all other codes are tracked. Versions that have less than 50 instances sending telemetry on any given data are filtered out to remove outliers. The response times of these web requests can be measured to estimate how satisfied a customer is with their service. 

### Apdex

Apdex (Application Performance Index) aims to convert measurements into insights about user satisfaction, by specifying a uniform way to analyze and report on the degree to which measured performance meets user expectations

It is calculated according to the following formula:


    Apdex = (SatisfiedCount + ToleratingCount * 0.5) / TotalCount
    
Unless being displayed in a more specific context (e.g. on individual data records, or group of records), the Apdex is calculated using:

- API (Web) Requests that return a 2xx response
- Exclusing certain requests that are called often and are cached (e.g. ServerStatus)
- A Satisified Threshold of <= 50ms
- A Tolerating Threshold of > 50ms and <= 200ms

Apdex has a range of 0-100. This means that there is a uniform scale to evaluate customer experience. A higher number indicates a more positive user experience. These thresholds can be varied to experiment with Apdex scores given a certain appetite for tolerance. The examples in this blog display the default threshold values. They can be modified in real time in the application.

## Visualizing Apdex and Octopus Deploy

There are several ways to vizualise apdex. The following graphs are some of the ways we use Crow's nest to display apdex and gain useful insights.

### Apdex for cloud and deploy

The blue line shows the Apdex performance of recent versions in Octopus Cloud and the deployment server. The cloud performance has been consistent around 90 for this period. The deployment server is the internal Octopus instance. The orange graph indicates a major dip in Apdex from 2021.2.2048 where it recovered in 2021.2.4155. The causes of this crash and recovery are known. It can be useful to look back and visually see how different versions affected the user experience. If any large dip in performance is identified, a root cause analysis can be conducted where causes are identified and addressed.

![Apdex Cloud and Deploy](apdex-cloud-deploy.png "Apdex Cloud and Deploy")

### Apdex by version and license

These set of graphs show the apdex score for different versions and licenses. The licenses are Cloud, On Premise, Trial, and Overall.  The first graph indicates that the trial subscription has suffered a slight dip in Apdex performance between  2020.3 and 2020.4 and again between 2020.5 and 2020.6. On the x.y level, the cloud and on premise licenses appear to be the most consistent, with the trial license suffering from more dips in performance. Overal, all licenses have Apdex levels around 80 or above which indicates good performance.

The apdex scores can be filtered at an x.y.z and license level. The orange, blue and purple graph show the on premise, cloud and trial license. This confirms that the cloud license was the most consistent license, maintaining a value of around 90 over multiple versions. The on premise license displayed higher peaks along with more frequent dips than the cloud license. The trial license experienced significant troughs which lowered its average apdex score. This indicates that the on premise and trial licenses have been more prone to performance fluctuations across different versions. 

An admin can look at these graphs and conduct a usability analysis betweent the cloud, on premise and trial versions. The goal would be to bring up the usability of the on premise and trial licenses to match the cloud license.

![Apdex by Version](apdex-by-version.png "Apdex by Version")

![Apdex by Version](apdex-by-version-z-onprem.png "Apdex by Version")

![Apdex by Version](apdex-by-version-z-cloud.png "Apdex by Version")

![Apdex by Version](apdex-by-version-z-trial.png "Apdex by Version")


### Apdex customer view

![Apdex Customer View](apdex-customer.png "Apdex Customer View")

### Apdex overall score

Each customer is given an overall apdex score. This can be linked to a threshold level. At apdex scores below a certain level, resources are depoloyed to identify and fix any faults causing the poor score.

![Apdex Score](apdex-score.png "Apdex Score")

### Apdex routes

Every web request has an endpoint. The endpoint is the subject of the request. The path to fulfil these requests are known as routes. The performance of individual routes can be seen in a customers view. This shows metrics such as the mean, median and highest value of a request. These metrics can be used to idenfiy the worst performing routes in a later version.

![Apdex Routes](apdex-route.png "Apdex Routes")

#### Apdex route difference

Crow's nest can be used to view the differences in routes across different versions. The project routes have been improved between 2020.6 and 2021.1.

![Apdex Routes Difference](apdex-route-diff.png "Apdex Routes Difference")

#### Apdex route view

Individual routes can be viewed by historical performance. This is useful when assessing whether updated versions of Octopus Deploy have improved or worsened the route. The performance of the certificates route above degradated from 2020.2 to 2020.6. The apdex score decreased from 62 to 16. In 2021.1 this route was inmproved leading to an apdex score of 77.

![Apdex Route View](apdex-route-view.png "Apdex Route View")

## Application

Telemetry is a powerful tool that can enable businesses to gain a complete picture over user experiences. Crows nest and Apdex gives full visibility of each user and each route. Performance can be compared historically across different versions and licenses. This allows the effect of each update to be assessed against another. Poor performing routes can be identified on a per user basis or across the Octopus Deploy platform.

