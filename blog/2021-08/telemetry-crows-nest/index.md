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

## Introduction

## Telemetry

Telemetry is the process of collecting measurements or statistical data and forwarding it to IT systems in a remote location. Many software companies use it to gather data on how their customers use and experience their product.

### Crows Nest

Octopus Deploy has implemented a telemetry tool that tracks how well customers are served web requests. A web request is sent when a customer wants to visit a page or poll an endpoint to get a response. The response times of these web requests can be calculated to estimate how satisfied a customer is with their service.  Web requests over several Octopus Deploy versions were tracked. The following categories are sent:

- Web request duration (2020.4)
- Database operation duration (2020.4)
- This should not happen (2021.1)
- Background job duration (2021.2)

Only web requests with a 2xx status are used in the calculations, however all other codes are tracked. Versions that have less than 50 instances sending telemetry on any given data are filtered out to remove outliers.

### Apdex

Apdex (Application Performance Index) aims to convert measurements into insights about user satisfaction, by specifying a uniform way to analyze and report on the degree to which measured performance meets user expectations

It is calculated according to the following formula:


    Apdex = (SatisfiedCount + ToleratingCount * 0.5) / TotalCount
    
Unless being displayed in a more specific context (e.g. on individual data records, or group of records), the Apdex is calculated using:

- API (Web) Requests that return a 2xx response
- Exclusing certain requests that are called often and are cached (e.g. ServerStatus)
- A Satisified Threashold of <= 50ms
- A Tolerating Threashold of > 50ms and <= 200ms

Apdex has a range of 0-100. This means that there is a uniform scale to evaluate customer experience

## Visualizing Apdex and Octopus Deploy

![Apdex Cloud and Deploy](apdex-cloud-deploy.png "Apdex Cloud and Deploy")

This graph shows the Apdex performance of recent versions in Octopus Cloud and the deployment server. The cloud performance has been consistent around 90 for this period, however the figure is skewed to include all light instances with less than 10 targets.

The deployment server is the internal Octopus instance. The graph indicates a major dip in Apdex from 2021.2.2048 where it recovered in 2021.2.4155. The causes of the crash and recovery are known. It can be useful to look back and visually see how different versions affected the user experience.

![Apdex by Version](apdex-by-version.png "Apdex by Version")

![Apdex by Version](apdex-by-version-z-onprem.png "Apdex by Version")

![Apdex by Version](apdex-by-version-z-cloud.png "Apdex by Version")

![Apdex by Version](apdex-by-version-z-trial.png "Apdex by Version")

The first graph displays the average value for different versions of Octopus Deploy and for different licenses. The licenses are Cloud, On Premise, Trial, and Overall. These values can be filtered to show only specific versions or licenses. The second graph shows versions at an x.y.z level and for the on premise license.

This graph can be used to compare the user experience of the different subscriptions across licenses. The trial subscription has suffered a slight dip in Apdex performance between the last few versions. The cloud and on premise version appear to be more consistent over versions

![Apdex Customer View](apdex-customer.png "Apdex Customer View")

![Apdex Score](apdex-score.png "Apdex Score")

![Apdex Routes](apdex-route.png "Apdex Routes")

![Apdex Route View](apdex-route-view.png "Apdex Route View")

## Application