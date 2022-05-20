---
title: Best practices for CI/CD
description: A brief summary of the post, 170 characters max including spaces.
author: terence.wong@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: placeholderimg.png
bannerImage: placeholderimg.png
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags:
  - DevOps
---

<!-- see https://github.com/OctopusDeploy/blog/blob/master/tags.txt for a comprehensive list of tags -->

Continuous integration and deployment (CI/CD) are practices that take software development from code to a live product. CI/CD forms part of DevOps practices, and there are many commonly agreed-upon best practices that users can follow to improve their deployment pipeline. Octopus Deploy is a deployment tool that supports the continuous deployment side of CI/CD, and we provide a best-in-class product that makes complex deployments easier. Here are eight best practices that we believe will help you in your deployment journey.

## Adopt agile methodologies

Agile methodologies are a vital component of CI/CD and DevOps. Agile methodologies is a project management approach that involves continuous collaboration with stakeholders and continuous improvement at each stage of the deployment process. The principle of agile methodologies is to have frequent feedback through small development iterations so that developers can closely align the final product with the product owner's needs. Agile methodologies contrast traditional waterfall methods, where projects were scoped and delivered in a single phase.

To get the most out of a CI/CD pipeline, software projects should be managed with the agile methodology so that the continuous feedback loop can improve the product.

## Use version-controlled code, connected to the deployment process, committed frequently

Developers should keep code created for a software project in a version-controlled system like Git. Version-controlled code allows a complete history and rollback of code to previous versions. Developers can also resolve conflict by using the merging methods of Git.

When using version control in a software project, committing a code change should trigger a CI/CD pipeline build. This trigger allows developers to test and validate changes to the codebase earlier.

Once a code change is set up to trigger an automated build,  developers should be encouraged to commit their code daily. Daily commits trigger automated tests more frequently and allow developers to notice any errors sooner.


## Use configuration as code for your deployment process

Configuration as Code (CaC) represents your deployment process in a Git-native system. Deployments inherit all the benefits of Git, such as branching, version control, and approvals as pull requests. In 2022 Q1, we released our general CaC release for Octopus Deploy, and we believe we have set an industry standard for CaC implementations. CaC implementations sacrifice usability for functionality. With CaC in Octopus Deploy, users get all the features of CaC whether they are using the UI or the version-controlled implementation.

CaC allows users to store their deployment process in Git. Changes to deployments can be tested in a branch and validated through a pull request. Git-native deployments make it easier to transfer a deployment set up from one environment to another. Consider using CaC in your deployment process today!

## Choose a tool that enables you to keep builds green

A green build in a CI/CD pipeline means that every test has passed and that the release has progressed to the next stage. Software teams aim to keep builds green. To help with that goal, software teams should choose a deployment tool that surfaces information to help keep builds green. Octopus Deploy has a UI that shows each release's deployment stage. The UI shows logs and error messages to help developers identify failing builds.

## Continuously automate your tests

Testing code changes is essential to producing reliable releases. The testing suite should cover all use cases for the product, from functional to non-functional tests. These tests should be automated so that a code change can trigger an automated test and build. Automated tests improve the agility of a software development project so that releases can be live faster.

## Strengthen the feedback loop through monitoring

The DevOps cycle incorporates feedback to improve the cycle for future iterations. Monitoring key system metrics can help diagnose a system for vulnerabilities and identify improvements. Telemetry data (logs, metrics, and traces) is data that developers can use to understand the system's internal state. Telemetry unlocks system observability by allowing developers to act on the data to fix the system.

## Use technologies that are fit-for-purpose

Every year there is a new flavor of the month technologies that people are saying will revolutionize the IT playing field. Whether it is containerization, machine learning, or blockchain, some technologies change the playing field, and others are too immature to make a real impact. When managing a CI/CD pipeline, it is essential only to choose technologies fit for purpose. While being cloud-native may make sense for some parts, forcing everything onto the cloud may not be the right solution. Adoption of new technologies can bring significant improvements, but taking a measured approach will avoid unnecessary pain when the costs of adoption outweigh the benefits.

## Take security seriously

When dealing with propriety software, security is a concern. The practices of a development pipeline should incorporate a security strategy. Many cloud providers like AWS, Azure, or Google have built-in security features such as IAM, secrets, and role-based permissions. Many customers are rightly concerned with security, and companies should look to invest in certifications such as ISO 27001 and SOC II.

## Conclusion

CI/CD is part of the DevOps methodology and helps bring software projects from code to a live product. I have listed eight best practices you can use to make the most of your CI/CD process. Octopus Deploy is a continuous deployment tool that can help your CI/CD goals.

Happy deployments!
