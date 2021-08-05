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

As Octopus Deploy has grown, it now covers a large software surface area. A given user may need to interface with many different touch points across Octopus Deploy. These may be viewing the dashboard, securely signing on through certificates or viewing the projects page. In order to serve our customers better, we want to be able to tell whether users accessing these touch points are receiving their content in a timely manner.

Telemetry is the process of collecting usage statistics and forwarding it to IT systems where it can be analyzed. Many software companies use it to gather data on how their customers use and experience their product. Octopus Deploy has implemented a telemetry tool named 'Crow's Nest' that tracks how quickly customers are served web requests. Telemetry is activated by default unless users choose to opt-out. 

The Crow's Nest symbolizes a tool that has a high level overview of the way users are experiencing the product.  A web request is sent when a customer wants to visit a page or poll an endpoint to get a response. Web requests over several Octopus Deploy versions are tracked.

Only web requests with a 2xx status are used in the calculations, however all other codes are tracked. A 2xx status indicates a successful web request. Versions that have less than 50 instances sending telemetry on any given data are filtered out to remove outliers. The response times of these web requests can be measured to estimate how satisfied a customer is with their service. 

### Apdex

Apdex (Application Performance Index) aims to convert measurements into insights about user satisfaction, by specifying a uniform way to analyze and report on the degree to which measured performance meets user expectations

It is calculated according to the following formula:


    Apdex = (SatisfiedCount + ToleratingCount * 0.5) / TotalCount
    
Unless being displayed in a more specific context (e.g. on individual data records, or group of records), the Apdex is calculated using:

- API (Web) Requests that return a 2xx response
- Excluding certain requests that are called often and are cached (e.g. ServerStatus)
- A Satisfied Threshold of <= 50ms
- A Tolerating Threshold of > 50ms and <= 200ms

Apdex has a range of 0-100. The scale is:
- 94-100 : excellent
- 85-94 : good
- 70-85 : fair
- below 70 : poor
- below 50 : unacceptable

This means that there is a uniform scale to evaluate customer experience. A higher number indicates a more positive user experience. These thresholds can be varied to experiment with Apdex scores given a certain appetite for tolerance. The examples in this blog display the default threshold values. They can be modified in real time in the application to view Apdex scores against different criteria.
 
## Visualizing Apdex and Octopus Deploy

There are several ways to visualize Apdex. The following graphs are some of the ways we use Crow's nest to display Apdex and gain useful insights.

### Apdex for cloud and deploy

The blue line shows the Apdex performance of recent versions in Octopus Cloud and the deployment server. The cloud Apdex performance has been consistent around 90 for this period. The deployment server is the internal Octopus instance not released to customers. The orange graph indicates a major dip in Apdex from 2021.2.2048 where it recovered in 2021.2.4155. The causes of this crash and recovery are known. It can be useful to look back and visually see how different versions affected the user experience. If any large dip in performance is identified, a root cause analysis can be conducted where causes are identified and addressed.

![Apdex Cloud and Deploy](apdex-cloud-deploy.png "Apdex Cloud and Deploy")

### Apdex by version and license

These set of graphs show the Apdex score for different versions and licenses. The licenses are cloud, on premise, trial, and overall.  The first graph below indicates that the trial subscription has suffered a slight dip in Apdex performance between  2020.3 and 2020.4 and again between 2020.5 and 2020.6. On the x.y level, the cloud and on premise licenses appear to be the most consistent, with the trial license suffers from more dips in performance. Overall, all licenses have Apdex levels around 85 or above which indicates good performance.

The Apdex scores can be filtered at an x.y.z and license level. The orange, blue and purple graph below show the on premise, cloud and trial license. This confirms that the cloud license was the most consistent license, maintaining a value of around 90 over multiple versions. The on premise license displayed higher peaks along with more frequent dips than the cloud license. The trial license experienced significant dips which lowered its average Apdex score. This indicates that the on premise and trial licenses have been more prone to performance fluctuations across different versions. 

An admin looks at these graphs and conducts a usability analysis between the cloud, on premise and trial versions. The goal would be to bring up the usability of the on premise and trial licenses to match the cloud license. The latest results show all versions have an average score of around 85 and are close to one another which is a good result.

![Apdex by Version](apdex-by-version.png "Apdex by Version")

![Apdex by Version](apdex-by-version-z-onprem.png "Apdex by Version")

![Apdex by Version](apdex-by-version-z-cloud.png "Apdex by Version")

![Apdex by Version](apdex-by-version-z-trial.png "Apdex by Version")


<!--### Apdex customer view

![Apdex Customer View](apdex-customer.png "Apdex Customer View")-->

### Apdex overall score

Each customer is given an overall Apdex score. This can be linked to a threshold level. At Apdex scores below a certain level, resources are deployed to identify and fix any faults causing the poor score.

![Apdex Score](apdex-score.png "Apdex Score")

### Apdex routes

Every web request has an endpoint. The endpoint is the subject of the request. The path to fulfill these requests are known as routes. The performance of individual routes can be seen in a customers view. This shows metrics such as the mean, median and highest value of a request. These metrics can be used to identify the worst performing routes in a later version.

![Apdex Routes](apdex-route.png "Apdex Routes")

#### Apdex route difference

Crow's nest can be used to view the differences in routes across different versions. The project routes have been improved between 2020.6 and 2021.1 as indicated by the green difference indicators for the Apdex score

![Apdex Routes Difference](apdex-route-diff.png "Apdex Routes Difference")

#### Apdex route view

Individual routes can be viewed by historical performance. This is useful when assessing whether updated versions of Octopus Deploy have improved or worsened the route. The performance of the certificates route above degraded from 2020.2 to 2020.6. The Apdex score decreased from 62 to 16. In 2021.1 this route was improved leading to an Apdex score of 77.

![Apdex Route View](apdex-route-view.png "Apdex Route View")

Telemetry is a powerful tool that can enable businesses to gain a complete picture over user experiences. Telemetry and Apdex give full visibility of each user and the performance of each route. Performance can be compared historically across different versions and licenses. This allows the effect of each update to be assessed against another. Poor performing routes can be identified on a per user basis or across the Octopus Deploy platform. Combining visualizations of Apdex scores and data analysis, our Crow's Nest tool helps to continually improve the customer experience, leading to customer satisfaction.

