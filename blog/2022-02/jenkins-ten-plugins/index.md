---
title: 10 of our favorite Jenkins plugins
description: Jenkins has over 1800 community-created plugins to help with continuous integration. Here are 10 we think are useful, as part of our series about CI and build servers.
author: andrew.corrigan@octopus.com
visibility: public
published: 2022-02-23-1400
metaImage: blogimage-jenkinsconfigurationtop10plugins-2022.png
bannerImage: blogimage-jenkinsconfigurationtop10plugins-2022.png
bannerImageAlt: Person working at a computer with plugs connecting to it. When they connect, little stars appear around the connection.
isFeatured: false
tags:
  - DevOps
  - CI Series
  - Continuous Integration
  - Jenkins
  - Testing
---

As an open-source Continuous Integration (CI) platform, one of our favorite things about Jenkins is its strong community. Nowhere is this more evident than in the Jenkins Plugins Index.

There are over 1800 user-created plugins in the Index, allowing you to extend Jenkins’ features and change your instance to meet your team’s needs.

Here are 10 of our favorite Jenkins plugins, what they can bring to your pipeline, and [how to install them](#install).

## 1: Blue Ocean

Jenkins’ own [Blue Ocean plugin](https://plugins.jenkins.io/blueocean/) freshens up the UI with a modern look and feel.

The simple, visual-focused design makes it easier to create pipelines, read process statuses, and spot pipeline problems. Plus, it allows you to create your own dashboards, so you only see what you need.

## 2: Simple Theme

With the [Simple Theme plugin](https://plugins.jenkins.io/simple-theme-plugin/), you can change the way Jenkins looks and feels using CSS and JavaScript.

You can also find readily available themes on GitHub via the [plugin’s Index page](https://plugins.jenkins.io/simple-theme-plugin/#plugin-content-themes).

## 3: Performance Publisher

Data isn't useful if you're not sure how to read it, and that’s what makes the [Performance Publisher](https://plugins.jenkins.io/perfpublisher/) plugin so helpful.

Performance Publisher helps you spot trends in your test results. It reads the XML files created by your testing tool and turns them into easily readable graphs and stats.

## 4: GitHub

The [GitHub plugin](https://plugins.jenkins.io/github/) connects Jenkins to GitHub. This allows for cross-functionality, such as:

-	Hyperlinks between the 2 services
-	Build status reports
-	Hook triggers
-	Manual and automatic modes

See [the plugin’s Jenkins Index page](https://plugins.jenkins.io/github/) for more information, including example pipelines.

## 5: Kubernetes

The [Kubernetes plugin](https://plugins.jenkins.io/kubernetes/) helps to automate scaling when you're running Jenkins in a Kubernetes cluster.

This plugin creates a pod for each dynamic agent that connects automatically via variables. One container in the cluster acts as the Jenkins agent, allowing you to run whatever jobs or processes you need on the others.

## 6: Dashboard View

The [Dashboard View plugin](https://plugins.jenkins.io/dashboard-view/) lets you create new views and portals in Jenkins. You build your dashboard from a range of portlets, to help you track jobs, tests, and more in digestible ways.

## 7: Maven Integration

As a popular build automation tool for Java projects, the [Maven plugin](https://plugins.jenkins.io/maven-plugin/) offers new features for those running Apache’s Maven in their pipeline.

The plugin extends Jenkins’ compatibility to better support:

- The triggering of jobs and deployments after successful builds
- Auto setup of other [reporting plugins](https://plugins.jenkins.io/ui/search?sort=relevance&categories=&labels=report&view=Tiles&page=1&query=)

## 8: Folders

The [Folders plugin](https://plugins.jenkins.io/cloudbees-folder/) is so useful, Jenkins recommends it during setup. This simple plugin helps you sort your instance by letting you group jobs in nestable folders.

You can give each folder its own dedicated view depending on its purpose, helping to clean up the clutter in the UI.

## 9: Bootstrapped-multi-test-results-report

The [bootstrapped-multi-test-results-report](https://plugins.jenkins.io/bootstraped-multi-test-results-report/) is a visual tool that lets you make in-depth HTML reports of test results. The plugin supports the likes of Cucumber, Junit, RSpec, and TestNG.

Check out an [example report on the plugin’s website](https://web-innovate.github.io/cucumber-reports/featuresOverview.html).

## 10: Pipeline Utility Steps

The [Pipeline Utility Steps plugin](https://plugins.jenkins.io/pipeline-utility-steps/) offers many extra steps to help with the smaller things in your Jenkins pipeline.

With this plugin you can:

- Find, manage, and create files
- Read and write common config files including CSV, JSON, and YAML
- Zip and unzip files
- Read and write Maven Model data structures
- Compare 2 version numbers

Check out the plugin’s GitHub page for the [full list of steps and the syntax needed to use them](https://github.com/jenkinsci/pipeline-utility-steps-plugin/blob/master/docs/STEPS.md).

## Bonus recommendation: Octopus Deploy

It wouldn't be right if we didn't mention our own [Octopus Deploy plugin](https://plugins.jenkins.io/octopusdeploy/), which connects Octopus to your Jenkins pipeline.

Once Jenkins has finished compiling, testing, and packaging your code, our plugin can:

- Automatically trigger deployments in Octopus when a build completes
- Fail a build in Jenkins if the deployment in Octopus fails

See our documentation for more on [using Jenkins with Octopus](https://octopus.com/docs/packaging-applications/build-servers/jenkins). Also, check out some of our other Jenkins-related blogs:

-	[Octopus plugin for Jenkins: Painless Jenkins integration](https://octopus.com/blog/octopus-jenkins-plugin)
-	[Using Jenkins Pipeline with Octopus](https://octopus.com/blog/using-jenkins-pipelines)
-	[Deploying to Octopus from Jenkins using Pipelines](https://octopus.com/blog/deploying-to-octopus-from-jenkins)

If you're not already using Octopus Deploy, you can [sign up for a free trial](https://octopus.com/start).

## How to install Jenkins plugins {#install}

You can install plugins with either the Jenkins web UI or the [Jenkins CLI (command-line interface)](https://www.jenkins.io/doc/book/managing/cli/) using the `install-plugin` command.

To install Jenkins plugins, you must:

- Configure the Jenkins controller to allow meta-data downloads from an update center (configured during Jenkins setup)
- Be an admin of your Jenkins instance

Before you install a plugin, read through its documentation in full. Jenkins will install all dependency plugins during an installation, but make sure to note any other prerequisites or extra steps.

See Jenkins’ documentation for more detail on managing and installing plugins, including:

-	[Advanced install techniques](https://www.jenkins.io/doc/book/managing/plugins/#advanced-installation)
-	[Disabling and uninstalling plugins](https://www.jenkins.io/doc/book/managing/plugins/#disabling-a-plugin)

### Jenkins web UI

To search for and install plugins from the Jenkins web UI:

1. Click **Manage Jenkins** from the left menu and select **Manage Plugins**.
1. Click the **Available** tab and use the search box to find plugins from the update center.
1. Tick the box to the left of the needed plugin and click **Install without restart** or **Download now and install after restart**. You can install most plugins without a restart.

### Jenkins CLI

Use the following command-line to install plugins with the Jenkins CLI. Replace `[SOURCE]` with the local file, URL, or update center to install from:

```
java -jar jenkins-cli.jar -s http://localhost:8080/ install-plugin [SOURCE]
```

You can also use these modifiers at the end of the command-line:

-	`-deploy`: install the plugin without a reboot
-	`-name VAL`: add a short name (by default Jenkins takes the plugin’s name as it appears in the source)
-	`-restart`: restart Jenkins once the plugin installs successfully

## What's next?

These are some of our favorite plugins to help you with your Jenkins pipeline, but we’re only scratching the surface. There are many more to find on the [Jenkins Index](https://plugins.jenkins.io/) that can further aid you in your CI efforts.

!include <jenkins-free-tool>
  
!include <jenkins-webinar-jan-2022>
  
!include <q1-2022-newsletter-cta>

Happy deployments! 

!include <related-content>
