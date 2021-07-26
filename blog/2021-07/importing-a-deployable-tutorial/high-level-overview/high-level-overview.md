---
title: Getting Started
description: An overview and roadmap to learning Octopus Deploy.
position: 0
---

## Welcome!

If you are reading this, then you have successfully deployed your first application through Octopus Deploy! This overview is meant to provide some additional context and information about Octopus deploy as well as point you in the right direction to learn more.

## What is Octopus Deploy and who is it for?

Octopus Deploy was founded in 2012 to create happy deployments, and by extension, happy software teams. Octopus Deploy focuses on three main parts of continuous delivery: Deploy, Release and Operate. The Octopus Deploy product helps to automate deployments, safeguard their deployments for release, and automate the tasks required outside the deployment window.

The market model of Octopus Deploy follows a bottom-up approach, meaning our product is targeted at the software engineering teams using the product. We find that a developer-led adoption model means that the product is targeted at the end users who then convince decision makers to adpopt the product company wide.

The ideal target market of Octopus Deploy is medium to large sized companies. Typically a company with 200 or more employees is the tipping point as it is large enough to warrant a complex deployment setup for Octopus Deploy to help with.

Octopus Deploy is a continous delivery software that natively integrates with several other software tools. This is essential as Octopus Deploy sits in the middle of the CI/CD toolchain, orchestrating deployments from release to deployment. Octopus Deploy acheives best in class tooling by choosing to do a smaller number of things at high quality, over trying to do everything.

tl;dr

:::hint Octpus Deploy provides best in class automated continous delivery for cloud and on premise for medium to large businesses.:::

## What have I learned?

In this tutorial, you have imported an exported project and deployed a package to an Azure web application. This application is now hosted in Azure. Through setting up this page, you have been exposed to some of the fundamental concepts of Octopus Deploy such as:

 - Deployment targets
 - Environments
 - Accounts
 - Packaging applications
 - Projects
 - Releases
 - Deployments
 - Variables
 - Runbooks

The blog post represented a simple example. Production level solutions are more complex with many deployment steps and deployment targets. Ther are also other advanced Octopus Deploy features such as tenants and the Octopus REST API that developers can use. A comprehensive documentation of all Octopus Deploy features is found [here](https://octopus.com/docs).

## How does this help me?

With this knowledge, you are now able to utilize the Octopus Deploy platform to serve your continuous deployment needs. We have a wide range of example use cases that our customers have used Octopus Deploy for. It is likely that some or a mixture of these apply to you. Find out more [here](https://octopus.com/company/customers). 

The customers that have signed up with us usually experience an 'a-ha!' moment where they understand the value proposition that Octopus Deploy provides and how it can solve their problems. Through the resources contained in this overview, and the resources that you will be pointed to, we want hope you can understand whether Octopus Deploy can meet your needs.

## How much does it cost?

Octopus is free for up to 10 deployment targets or less. This applies to both the Octopus Cloud and self-hosted versions. For deployments that require more targets than this, there is a pricing guide that can be found [here](https://octopus.com/pricing/overview). We have found that there are definitive use cases for the free tier, some customers have used the free tier to orchestrate their larget complex deployment that uses the paid license. 

## Where can I learn more?

Octopus Deploy can be a lot to take in for a new user. As it is a product that sits in between several other third party tools, learning Octopus also requires learning other tools and how it integrates with Octopus. Fortunately, there are plenty of resources to learn more about Octopus Deploy and its integtrations.

### Introdution to Octopus Deploy

<iframe width="560" height="315" src="https://www.youtube.com/embed/Z77T3SHRLKE" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### YouTube

We have a channel where you will where find getting started resources and guides. It also contains Q&As, release notes and general information about Octopus Deploy practices.

<iframe width="560" height="315" src="https://www.youtube.com/embed/KLWFcETK4n4" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### Webinars

Octopus runs a series of webinars to cover fundamental and more advanced topics. Ocotopus integrates with several cloud service providers and tools. Often there will be webinars dedicated to highlighting the linkages between the two. For new users, this Octpus 101 webinar is useful:

<iframe width="560" height="315" src="https://www.youtube.com/embed/mo0D4d5hFFU" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### Other useful links

#### Blog 

We run a blog that contains a lot of useful information for new and existing users. Some topics we have covered are:

- [general advice or insights about continous integration](https://octopus.com/blog/difference-between-ci-and-cd)
- [specific solutions to common customer problems](https://octopus.com/blog/chocolatey-powershell-and-runbooks)
- [an explanation of the new features we are pushing out](https://octopus.com/blog/github-actions-for-octopus-deploy)
- [tool specific guides](https://octopus.com/blog/deploying-ruby)
- [an ebook about Kubernetes deployments](https://octopus.com/blog/10-pillars-kubernetes-deployments)


There are many other types of topics such as release notes, company announcements and more. We use the blog to create a relationship between Octopus Deploy and our customers. We find that our customers want to engage with the blog and find it useful and informative. You can find the full blog [here](https://octopus.com/blog).

#### Slack

Our Community Slack instance is a great place for our users to discuss tips and tricks and solve problems with the help of other Octopus Deploy users. A lot of Octopus team members respond there. You can join the Communicty Slack [here](https://octopus.com/slack).

#### Support

We have a dedicated support team that is here to answer any questions you may have. They are constantly updating they knowledge to support not only the Octopus Deploy product, but the software tools that integrate and work with Octopus Deploy. Their goals is to give accurate and timely answers to all support questions. Find them [here](https://octopus.com/support).


#### Demos & Samples

We have hosted some demo and sample instances of Octopus Deploy:

- [this is a demo version of Octopus Deploy](https://demo.octopus.com/)
- [this is an instance with sample projects](https://samples.octopus.app/)

Use these instances as sandbox environments to play with once you have understood the basics. We encourage trial and experimentation. If there are any samples you would like to see, please let us know vis support or slack.

We hope that this overview has given you a useful overview for getting started with Octopus Deploy. Octopus Deploy covers a large software surface area and we want the new user experience to be beneficial but also effective. We want to to accurately evaluate whether Octopus Deploy is right for you, and if it is, we want to give you all the resources you require to solve your CI/CD problems faster. Our support team is always here to help and we look forward to helping you achieve your CI/CD goals!