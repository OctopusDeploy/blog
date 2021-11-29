---
title: 10 of our favorite Jenkins plugins
description: Jenkins has over 1800 community-created plugins to help with continuous integration. Here are 10 we think are useful!
author: andrew.corrigan@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags:
  - DevOps
  - Continuous Integration
  - Jenkins
  - Testing
---

As an open-source continuous integration (CI) platform, one of our absolute favorite things about Jenkins is the strong community batting for it. Nowhere is this more evident than in the Jenkins Plugins Index.

There are over 1800 user-created plugins in the Index, allowing you to extend Jenkins’ features and change your instance to meet your team’s needs.

So, here’s 10 of our favorite Jenkins plugins, what they can bring to your pipeline and [how to install them](#install).

## 1: Blue Ocean

Jenkins’ own [Blue Ocean plugin](https://plugins.jenkins.io/blueocean/) freshens up the UI with a modern look and feel.

The simpler, visual-focused design makes it easier to create pipelines, read process statuses and spot pipeline problems. Plus, it allows users to create their own dashboards, so they only see what they need.

## 2: Simple Theme

With the [Simple Theme plugin](https://plugins.jenkins.io/simple-theme-plugin/), you can change the way Jenkins looks and feels using CSS and JavaScript.

What’s more, you can find a handful of readily available themes on GitHub via the [plugin’s Index page](https://plugins.jenkins.io/simple-theme-plugin/).

## 3: Performance Publisher

Data is useless if you aren’t sure how to read it, and that’s what makes the [Performance Publisher](https://plugins.jenkins.io/perfpublisher/) plugin so helpful.

This plugin helps you spot trends in your test results. It reads the XML files created by your testing tool and turns them into easily readable graphs and stats.

## 4: GitHub

The [GitHub plugin](https://plugins.jenkins.io/github/), as you might expect, connects Jenkins to GitHub. This allows for cross-functionality, such as:

-	hyperlinks between the 2 services
-	build status reports
-	hook triggers
-	manual and automatic modes.

See [the plugin’s Jenkins Index page](https://plugins.jenkins.io/github/) for more information, including example pipelines.

## 5: Kubernetes

The [Kubernetes plugin](https://plugins.jenkins.io/kubernetes/) helps to automate scaling when running Jenkins in a Kubernetes cluster.

This plugin creates a ‘pod’ for each dynamic agent that connects automatically via variables. One container in the cluster acts as the Jenkins agent, allowing you to run whatever jobs or processes you need on the others.

## 6: Dashboard View

The [Dashboard View plugin](https://plugins.jenkins.io/dashboard-view/) lets you create new views and portals in Jenkins. You build your dashboard from a range of ‘portlets’, all geared to help you track jobs, tests, and more in digestible ways.

## 7: Maven Integration

As a popular build automation tool for Java projects, this plugin offers a bunch of new features for those running Apache’s Maven in their pipeline.

The [Maven plugin](https://plugins.jenkins.io/maven-plugin/) extends Jenkins’ compatibility to better support:

- the triggering of jobs and deployments after successful builds
- auto setup of other [reporting plugins](https://plugins.jenkins.io/ui/search?sort=relevance&categories=&labels=report&view=Tiles&page=1&query=).

## 8: Folders

A simple plugin that's so useful, Jenkins recommends it during setup! The [Folders plugin](https://plugins.jenkins.io/cloudbees-folder/) helps you sort your instance by letting you group jobs within nestable folders.

You can give each folder its own dedicated view depending on its purpose, helping to clean up the clutter in the UI.

## 9: Bootstrapped-multi-test-results-report

The [bootstrapped-multi-test-results-report](https://plugins.jenkins.io/bootstraped-multi-test-results-report/) is a visual tool that lets you make in-depth HTML reports of test results. The plugin supports the likes of Cucumber, Junit, RSpec and TestNG.

Check out an [example report on the plugin’s website](https://web-innovate.github.io/cucumber-reports/featuresOverview.html).

## 10: Pipeline Utility Steps

The [Pipeline Utility Steps plugin](https://plugins.jenkins.io/pipeline-utility-steps/) offers many extra steps to help with the smaller things in your Jenkins pipeline.

With this plugin you can:

- find, manage, and create files
- read and write common config files including CSV, JSON and YAML
- zip and unzip files
- read and write Maven Model data structures
- compare 2 version numbers.

Check out the plugin’s GitHub page for the [full list of steps and the syntax needed to use them](https://github.com/jenkinsci/pipeline-utility-steps-plugin/blob/master/docs/STEPS.md).

## Bonus recommendation: Octopus

Well, um, this is embarrassing! But we’d be silly not to mention our own [Octopus Deploy plugin](https://plugins.jenkins.io/octopusdeploy/), which connects Octopus to your Jenkins pipeline.

Once Jenkins has finished compiling, testing, and packaging your code, our plugin can:

- automatically trigger deployments in Octopus when a build completes
- fail a build in Jenkins if the deployment in Octopus fails.

See our documentation for more on [using Jenkins with Octopus](https://octopus.com/docs/packaging-applications/build-servers/jenkins). Also, why not also check out some of our other Jenkins-related blogs:

-	[Octopus plugin for Jenkins: Painless Jenkins integration](https://octopus.com/blog/octopus-jenkins-plugin)
-	[Using Jenkins Pipelines with Octopus](https://octopus.com/blog/using-jenkins-pipelines)
-	[Deploying to Octopus from Jenkins using Pipelines](https://octopus.com/blog/deploying-to-octopus-from-jenkins)

## How to install Jenkins plugins {#install}

You can install plugins with either the Jenkins web UI or the [Jenkins CLI (command line interface)](https://www.jenkins.io/doc/book/managing/cli/) using the `install-plugin` command.

To install Jenkins plugins, you must:

- configure the Jenkins controller to allow meta-data downloads from an update center (configured during Jenkins setup)
- be an admin of your Jenkins instance.

Before you install a plugin, read through its documentation in full. Jenkins will install all dependency plugins during an install, but make sure to note any other prerequisites or extra steps needed.

See Jenkins’ documentation for more detail on managing and installing plugins, including:

-	[advanced install techniques](https://www.jenkins.io/doc/book/managing/plugins/#advanced-installation)
-	[disabling and uninstalling plugins.](https://www.jenkins.io/doc/book/managing/plugins/#disabling-a-plugin)

### Jenkins web UI

To search for and install plugins from the Jenkins web UI:
1. Click **Manage Jenkins** from the left menu and select **Manage Plugins**.
2. Click the **Available** tab and use the search box to find plugins from the update center.
3. Tick the box to the left of the needed plugin and click **Install without restart** or **Download now and install after restart**. You can install most plugins without a restart.

### Jenkins CLI

Use the following command line to install plugins with the Jenkins CLI. Replace [SOURCE] with the local file, URL, or update center to install from:

```
java -jar jenkins-cli.jar -s http://localhost:8080/ install-plugin [SOURCE]
```

You can also use these modifiers at the end of the command line:

-	-deploy: install the plugin without a reboot
-	-name VAL: add a short name (by default Jenkins takes the plugin’s name as it appears in the source)
-	-restart: restart Jenkins once the plugin installs successfully.

## What next?

These are just some of our favorite plugins to help you with your Jenkins pipeline, but we’re only scratching the surface. There are heaps more to find on the [Jenkins Index](https://plugins.jenkins.io/) that could further aid you in your CI efforts and more.

Happy deployments! 