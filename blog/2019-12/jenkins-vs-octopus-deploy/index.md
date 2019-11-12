---
title: Jenkins vs Octopus Deploy
description: Jenkins and Octopus Deploy are popular tools in the CI/CD and DevOps ecosystem 
author: rob.pearson@octopus.com
visibility: private
published: 2019-12-02
metaImage: octopus-2019.11-release-image.png
bannerImage: octopus-2019.11-release-image.png
tags:
 - Product
---

[Jenkins](https://jenkins.io) and [Octopus Deploy](https://octopus.com/) are popular DevOps automation tools that help teams build CI/CD pipelines to ship their software to customers. In this post, we'll 

## What is Jenkins? 

`<TODO: Jenkins dashboard and project>`

`<Tone of this section is 100% factual. State facts about each tool in without flare. More about what it does here. How and why are in the Jenkins vs Octopus section below.>`

<TODO: Merge the two options.>

> Jenkins is the leading open source automation server supported by a large and growing community of developers, testers, designers and other people interested in continuous integration, continuous delivery and modern software delivery practices. Built on the Java Virtual Machine (JVM), it provides more than 1,500 plugins that extend Jenkins to automate with practically any technology software delivery teams use. In 2019, Jenkins surpassed 200,000 known installations making it the most widely deployed automation server.
- https://github.com/cdfoundation/faq

Jenkins is a free, open source automation server with a vibrant community and a vast repository of plugins. Jenkins enables teams to automate the build, testing and even deployment of their software. It's core is a script execution/orchestration engine and it unlocks the ability to run any task that can be scripted and executed.

### Jenkins Benefits

Jenkins brings great benefits to the 

`<Tone: Positive facts but some opinion is OK>`

- Open source 
- Free and commercial support options
- 1500+ plugins
- Huge community and user base
- Infrastructure as Code
- Blue ocean user interface

`<TODO: Figure out how to present weaknesses. i.e. Primitive interface. Hard to model deployment through environments etc.>`

__Weaknesses:__
* User interface is bare-bones
* <Custom scripts require maintenance>
* <Advanced features require custom scripting and maintain>
* Large plugin library but mix of modern up-to-date plugins and abandoned out-of-date ones

## What is Octopus Deploy? 

`<TODO: Image of a deployment process and dashboard>`

`<Tone of this section is 100% factual. State facts about each tool in without flare. More about what it does here. How and why are in the Jenkins vs Octopus next section?`

Octopus Deploy is a DevOps tool focused on release management, automating deployments and executing operations runbooks. 
Octopus enables teams to define their deployment process and deploy releases to all of their infrastructure consistently. It allows teams to define environments and deploy/promote releases through those environments. 

Or something like this ... 

Octopus Deploy is focused on three things. 

1. Managing releases
2. Automating deployments
3. Operating applications

### Octopus Deploy Benefits

Octopus brings numerous benefits to the world of DevOps tooling.

- Free starter license for small teams
- Self-hosted, or in the Cloud

**Release Management**

- Better visibility with our deployment dashboards
- Promote releases with confidence thanks to release shapshots so you use the exact same deployment process when you're ready to deploy to production <- needs work
- Approvals w/ Manual Interventions when you need a person to approve it. 

**Deployment automation**

- Consistent deployment process with 300+ deployment steps
- Update your configuration files with scoped variables
- Environments
- Projects deployment process, variablesani
- Advanced patterns: rolling deployments, blue-green deployments

**Operations runbooks**

- 




- Infrastructure as Code w/ official Terraform provider

__Weaknesses:__
* Octopus has a free edition but it's a commercial product
* Mix of open source and closed source components
* Another tool in your toolchain

## Jenkins vs Octopus Deploy

Jenkins and Octopus are complimentary and can be used together to build a world-class CI/CD pipeline.

* Benefit 1 of using them together 
* Benefit 2 of using them together 
* Benefit 3 of using them together 

### Benefit 1

Octopus has an official jenkins plugin to make it easy to itegrate both tools and take advantage of both tools. 

### Benefit 2

`<TODO>`

### Benefit 3

`<TODO>`

## Conclusion

Jenkins and Octopus Deploy work extremely well together leveraging the Octopus Jenkins plugin. 

- Use Jenkins to build and test your application and then push your build artifacts to Octopus.
- Use Octopus Deploy to automate your deployments and promote releases through your environments like Dev, Test and Production.
- Use Operations Runbooks in Octopus Deploy to keep your applications running smoothly to handle emergencies as well as routine maintenance.
