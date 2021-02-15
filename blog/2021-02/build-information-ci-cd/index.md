---
title: Use build information to get better visibility across your CI/CD pipeline from build to deployment
description: Learn how to to include build information from your CI server in your CD processes. 
author: shawn.sesna@octopus.com 
visibility: private
published: 2022-02-17
metaImage: 
bannerImage: 
tags:
---

Continuous Integration (CI) typically involves three components: a source control server, issue tracking, and a build server.  Tools such as Azure DevOps combine all components into a single solution whereas other configurations have them separated such as using GitHub for source control, TeamCity for build, and Jira for issue tracking.  When it comes to Continuous Delivery (CD), commits and isue tracking are extremely important to ensure the correct version of software is deployed.  The Octopus Deploy product employs a feature called [Build Information](https://octopus.com/docs/packaging-applications/build-servers#build-information) which allows you to tie versions of your software to specific commits and isues.  

It is important to note that while commits will work in all cases, issue tracking will only function if you have one of the following integrations configured in Octopus Deploy
- Azure DevOps Issue Tracking
- GitHub Issue Tracking
- Jira Integration

In this post, I'll walk you through configuring Octopus Depoloy to include build information and commits and how this information can be used during the deployment process.

## Setup
For this demonstration, I'll be using the second scenario described above: GitHub, TeamCity, and Jira, which is a fairly common configuration.  While I'll be addressing these specific technologies, the overall configuration will be similar regardless of which toolset you use.

## OctoPetShop
[OctoPetShop](https://github.com/OctopusSamples/OctoPetShop) is a fictitious pet store application written in .NET Core consisting of a web front-end, two services, and a database.  This post assumes you have some familiarity with Git and GitHub and will not cover project creation or initial check-in of code.

### Create a bug
We'll start off by logging a bug in Jira for the OctoPetShop application.  This post assumes you already know how to create a project and an issue within Jira.

![](jira-issue.png)
  
Once the issue has been created, we'll need to take note of the of the `key` value as we'll need this to correctly tag our commits.  In this case, the value is `OPS-1`.

![](jira-issue-ops-1.png)

