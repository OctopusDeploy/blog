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

Continuous integration and delivery (CI/CD) are practices that take software development from code to a live product. CI/CD forms part of DevOps practices, and there are many commonly agreed-upon best practices that users can follow to improve their deployment pipeline. 

If you work in DevOps, you would have used a build server like Jenkins and a deployment tool like Octopus Deploy to complete your deployment process. Octopus Deploy supports the continuous delivery side of CI/CD, and we provide a best-in-class product that makes complex deployments easier. 

At Octopus, we believe in the power of eight. Eight lessons for eight tentacles! Here are eight best practices that we believe will help you in your deployment journey.

## Adopt agile methodologies

Agile methodologies are a vital component of CI/CD and DevOps. Agile methodologies is a project management approach that involves continuous collaboration with stakeholders and continuous improvement at each stage of the deployment process. The principle of agile methodologies is to have frequent feedback through small development iterations so that developers can closely align the final product with the user needs. Agile methodologies contrast traditional waterfall methods, where projects were scoped and delivered in a single phase.

To get the most out of a CI/CD pipeline, software projects should be managed according to Agile and Lean principles so that the continuous feedback loop can improve the product. We have seen that agile methodologies usually starts off as a set of checklists to tick off for upper management. In the initial stages, agile is applied to meet the checklist. As teams have permission to explore the agile space, after a year or so, agile is running in earnest and the true benefits are seen.

## Use version-controlled code, connected to the deployment process, committed frequently

If you work in software, you will have heard about Git and probably use Git. The wars on source-controlled code have been fought and won, and Git is now synonymous with source-control. This best practice is an obvious one, but it is essential, you need to keep your code in source-control. Source-controlled code allows a complete history and rollback of code to previous versions. You can also resolve conflicts by using the merging methods of Git.

When using source control, committing a code change should trigger a CI/CD pipeline build. This trigger allows developers to test and validate changes to the codebase earlier. Once a code change is set up to trigger an automated build,  developers should be encouraged to commit their code at least once a day. Daily commits trigger automated tests more frequently and allow developers to notice any errors sooner.


## Use configuration as code for your deployment process

Configuration as Code (CaC) represents your deployment process in a Git-native system. Deployments inherit all the benefits of Git, such as branching, version control, and approvals as pull requests. In 2022 Q1, we released our general CaC release for Octopus Deploy, and we believe we have set an industry standard for CaC implementations. CaC implementations sacrifice usability for functionality. With CaC in Octopus Deploy, users get all the features of CaC whether they are using the UI or the version-controlled implementation.

CaC allows users to store their deployment process in Git. Changes to deployments can be tested in a branch and validated through a pull request. Git-native deployments make it easier to transfer a deployment set up from one environment to another. Consider using CaC in your deployment process today!

## Choose a tool that enables you to keep builds green

A green build in a CI/CD pipeline means that every test has passed and that the release has progressed to the next stage. Software teams aim to keep builds green. To help with that goal, software teams should choose a deployment tool that surfaces information to help keep builds green. Many deployment processes only use a build server that pushes releases into production. In practice, only using a build server makes it harder to manage a release between different deployment stages. Using a dedicated deployment tool gives you a dedicated management layer to keep builds green.

A build server does not contain the concept of deployment stages. Octopus Deploy separates a release into environments like Test, Dev, and Production. A release can exist at different versions in each stage. Our UI that shows each release's deployment stage and can transition releases between stages.  Our UI also shows logs and error messages to help developers identify failing builds.

## Continuously automate your tests

Testing code changes is essential to producing reliable releases. The testing suite should cover all use cases for the product, from functional to non-functional tests. These tests should be automated so that a code change can trigger an automated test and build. Automated tests improve the agility of a software development project so that releases can be live faster.

A survey by [Mabel on the state of testing in DevOps](https://www.dropbox.com/s/nnagymzdcnoswc6/Benchmark-Report-State-of-Testing-in-DevOps.pdf?dl=0) indicates that automated testing (at least 4-5 different types of tests) is key to customer happiness. The [2021 State of DevOps DORA Report](https://www.dropbox.com/s/xycst8qsxnpsieu/state-of-devops-2021.pdf?dl=0), continuous testing is an indicator of success, with elite performers who meet their reliability targets 3.7 times more likely to leverate continuous testing.

## Strengthen the feedback loop through monitoring

Telemetry data (logs, metrics, and traces) is data that developers can use to understand the system's internal state. Telemetry unlocks system observability by allowing developers to act on the data to fix the system. Once your system is overlayed with telemetry data, you can use observability tools to add monitoring capabilities to the system.

Monitoring key system metrics can help diagnose a system for vulnerabilities and identify improvements. It is important to understand what metrics are important to measure. In the DevOps community, [DORA metrics](https://cloud.google.com/blog/products/devops-sre/using-the-four-keys-to-measure-your-devops-performance) are commonly accepted as key metrics for the success of a deployment pipeline.

## Use technologies that are fit-for-purpose

Every year there is a new flavor of the month technologies that people are saying will revolutionize the IT playing field. Whether it is containerization, machine learning, or blockchain, some technologies change the playing field, and others are too immature to make a real impact. When managing a CI/CD pipeline, it is essential only to choose technologies fit for purpose. 

While being cloud-native may make sense for some parts, forcing everything onto the cloud may not be the right solution. Adoption of new technologies can bring significant improvements, but taking a measured approach will avoid unnecessary pain when the costs of adoption outweigh the benefits.

## Take security seriously

As software projects get larger, the security risk increases due to more data handling, more users and more dependencies. Your deployment process should have a security strategy.  Many cloud providers like AWS, Azure, or Google have built-in security features such as IAM, secrets, and role-based permissions. These features can be used to manage some security concerns. Many customers are rightly concerned with security, and companies should look to invest in certifications such as ISO 27001 and SOC II to certify their compliance with security regulations.

On the 12th of May 2021, The United States government released an executive order on Improving the Nation's Cybersecurity. The executive order requires all vendors of government software projects to produce a Software Bill of Materials (SBOM). The SBOMs details all the software components so that it can be screened for cybersecurity. If you want an example of how to produce a SBOM and attach it to your deployment process, we have created a free tool called Octopus Builder that helps with this.

## Conclusion

CI/CD is part of the DevOps model and helps bring software projects from code to a live product. If you work in DevOps and implement CI/CD, you want to make sure that you are following some industry standard best practices for your pipeline. To help, I have listed eight best practices you can use to make the most of your CI/CD process. There are many tools out that that can help you with CI/CD, from build servers, to deployment tools to monitoring tools. Octopus Deploy fits into CI/CD as a continuous deployment tool that helps make complex deployments easier.

Happy deployments!
