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

Collecting and storing data are often the main functions of web applications.  The collected data often needs to be analyzed and shown in graphical format to help make decisions.  This usually takes the form of a report.  In this part of my series, I will demonstrate how to include SQL Server Reporting Services reports using Octopus Deploy.

## Building the project
As of Visual Studio 2017, MSBuild can build .rtpproj files.  Projects that are created in older versions of Visual Studio will need to configure the build agents similar to my SSIS post.