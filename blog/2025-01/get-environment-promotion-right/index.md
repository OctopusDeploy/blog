---
title: Why it’s critical to get environment promotion right
description: Learn why environment promotion can be challenging, particularly as you start to deploy to Kubernetes. 
author: nikita.dergilev@octopus.com
visibility: public
published: 2025-01-07-1400
metaImage: blog-environment-promotion.png
bannerImage: blog-environment-promotion.png
bannerImageAlt: An image of an application being promoted across two environments
isFeatured: false
tags: 
  - DevOps
  - Kubernetes
---

Whether you're deploying to Kubernetes, or anywhere else, environment promotion isn't a new concept. However, it can get challenging to manage, particularly at scale. So what are those challenges, and how can you mitigate them?

This post explores environment promotion and why it can be tricky, particularly as you start to deploy to Kubernetes. We cover:

- What environment promotion is and the typical structure
- Why it’s so important
- How environment promotion ties into Continuous Delivery 
- When it becomes challenging
- Solutions for getting environment promotion right
- What environment promotion looks like for Kubernetes


## What is environment promotion? 

Environment promotion, or environment progression, isn't a new concept in software development. However, you can quickly face challenges as you scale if you haven’t set it up correctly. So let’s start with a definition. 

Environment promotion is the process of promoting an application or service across different environments, typically with increasing maturity. For example, you start by deploying to the development environment, then promote to staging for testing after you ensure everything is working fine. Then, you promote to production, where your users will use your software. 

The concept behind environment promotion is that every change happens via the promotion process with a combination of human approvals validating that code is ready for production or automation to run tests and deploy. Deployments must only happen via this promotion process, meaning no manual changes. 

## A typical environment structure 

Each type of environment should have consistent configurations, limitations on what you can deploy there, and access policies. You can set up whatever environment structure works for you and your team, but most applications have 3 environment types:

- Development
- Pre-production
- Production

### Development environments 

Development environments let developers run and debug their code in environments outside their local development machine. Development environments can run branch versions of your application, and developers can usually access the infrastructure of these environments easily.

### Pre-production environments

You probably know pre-production environments by names like staging and test, but they all share a common purpose. They're the step between development and production. These are where you make sure the application and deployment will run successfully in production, and so should mirror your production environments. These environments should only run code from the main branch, to prevent changes you don't want pushed to your production environment. 

### Productions environments 

The most important environment is the production environment, as this is where your application serves customers. By this stage, code has been automatically tested and deployed multiple times to reduce the risk of something going wrong. 

## Why the sequence in environment promotion is so important

Having a structure in place for your environment promotion means you can be confident in everything that goes to production. If you implemented strict rules for environment promotion, you know that what you deployed to production will work as planned. This is because you've tested the new version of your application and the pipeline itself (including any changes to it) in the pre-production environment.

Having confidence in what you released is important for a few reasons:

- **Critical for testing deployments** - The only way to ensure that a deployment will work as expected is to test it. This is important not only for code and components but also for the deployment process. The best way to test the production deployment process is to use the same process in test environments. This lets you find and fix issues earlier, leading to a shortened feedback loop and less wasted time.
- **Reduced risk of something going wrong** - The more you test a new version of an application before it goes to production, the less risk there is that something will go wrong and create downtime. Environment promotion includes testing and gates to reduce the risk of unexpected errors when it reaches customers.
- **Crisis management** - Being confident in your deployment processes also helps during emergencies. Relying on a trusted, well-tested, automated process is much better than trying to improvise. Using an improvised process in a high-pressure situation makes you more likely to introduce even more problems.

## Automating environment promotion with Continuous Delivery

Every software company has some form of environment and rules around promotion in place. Yet, environment promotion remains a challenge. 

Continuous Delivery (CD) and deployment automation help you increase deployment frequency and the quality of your promotions between environments. This, in turn, helps you improve your DORA metrics. Teams that perform well against DORA metrics are more likely to achieve better customer satisfaction, operational efficiency, and overall organizational performance.

If you don’t automate environment promotion, there's a higher risk of something going wrong in production because you haven’t tested it rigorously. This means it isn’t as easy to ship as frequently, so smaller changes accumulate, and a new release can introduce major changes to your application. This makes deployment time more uncertain, which usually needs more testing to resolve potential conflicts of different features introduced between releases. You might need a dedicated Quality Assurance (QA) team and change management team to help with the process of approvals. This can make releases even more expensive, which can reduce release frequency further. 

However, if you have environment promotion in place, you’re more confident in what you ship, which means you can introduce more incremental changes. These changes are easy to test and have a smaller impact, so you can deliver value to your customers sooner and reduce the risk of something breaking. 

Environment promotion helps increase deployment frequency even more when you can automate the process, which is where CD comes in. CD provides end-to-end automation of all the environments and tests. Ideally, as soon you commit new code, it gets deployed to production in minutes with zero manual effort.

## When environment promotion becomes challenging

Whatever method you choose to automate your environment promotion, it should be fairly straightforward for the first few applications or microservices if you don’t have many environments or compliance or security requirements. However, it gets tricky as you start to scale, as you'll likely scale in multiple directions simultaneously. 

First, the number of applications will grow. Depending on the size of your organization, you can end up with hundreds, if not thousands, of individual applications you need to deploy independently. Some might have multiple updates daily, while others will get updates once every few months. Even if you reduce the number of updates to help manage the scale, this leads to the risk that the deployments won’t work if they’re not fully automated. 

The number of environments will also grow. For example, you might need to test multiple code versions at once, so you’ll likely have many development environments. Eventually, you’ll need to provision them dynamically. You may also need to implement environments for specific customers, for early access, or special QA environments to test your software in different integration scenarios.

Environments will also sometimes have more than one target. This means you must consider how you deploy to a few targets under one environment, whether it's a Kubernetes cluster, a cloud service, a VM, or different geographic locations, branches, or clients. 

Finally, your pipeline will become increasingly complicated as you add sophisticated tests, notifications, and integrations to meet compliance and security requirements.

## How to address the challenges of environment promotion

There are a couple of options to automate environment promotion. You can write custom code to deploy your software to a given environment, run all the required checks, and then deploy to the next environment. Or, you can use a dedicated tool to automate the process. 

With generic tools, like Jenkins or GitHub Actions, that you can use for running any tasks, including Continuous Integration (CI) and Continuous Delivery (CD), you can expect to write more code, as these aren’t specialized. Using custom scripts gets difficult to manage at scale, and you often end up with too many scripts to keep track of. This can incur a huge extra cost supporting a complex deployment landscape. 

If you use a dedicated CD tool, it will likely have a built-in model for environments, deployment steps, and lifecycles, without needing custom code. Your CD tool should be able to choose the right sequence and promotion conditions for each app, automating environment promotion for you at any scale. Examples of these tools include Octopus Deploy and Codefresh.

## Environment promotion for Kubernetes 

Kubernetes is powerful, but it doesn’t make environment promotion easy. Kubernetes manifests combine cluster and app configuration, meaning you must manage both with one process. Kubernetes is also a declarative platform. Therefore, it’s eventually consistent. After deployment, you need to know if the cluster achieved the desired state. So how do you solve environment promotion with Kubernetes? 

Conceptually, environment promotion with Kubernetes isn’t different from environment promotion on any other platform. The difference is the tools we use: the extra capabilities and the limitations these tools introduce. As mentioned, Kubernetes is a declarative platform. The best way to deploy to Kubernetes is to create a configuration file describing your application and infrastructure parameters and apply it to a cluster. This approach is simple as configurations are easy to understand, and deployment is just one command. However, this isn't ideal for environment promotion, as you must create and maintain a version of your configuration file per environment with every new deployment.

### Tools to solve environment promotion for Kubernetes

To solve these problems, the Kubernetes community introduced tools like Helm and Kustomize. These tools let you create a configuration once and alternate it with variables or simple patches per environment or tenant. These solutions are not as simple and human-readable as the plain YAML configuration, but it’s a good trade-off to enable configuration templating.

The Kubernetes community also introduced Kubernetes-specific deployment tools like Argo CD and Flux CD. These tools continuously apply configurations (which could be plain YAML, Helm charts, or Kustomize manifests) from Git to a cluster, ensuring that a cluster always runs a configuration stored in Git. These tools follow a framework called GitOps.

Deploying with these tools involves organizing configuration files into folders, repositories, or branches. Each environment is represented by its configuration file(s), and a deployment tool synchronizes these files with the corresponding cluster.

The big unsolved problem here is deployment automation. To deploy a new version of an application, we need to update a corresponding configuration file with the new app parameters (like a container image tag). We need to update these files according to the order of our environment promotion. We also still need to run tests, send notifications, perform policy checks, update change management tools, and take any other steps we'd typically take to promote an application to an environment.

There are multiple ways to solve this problem for deployments to Kubernetes. As with deployments in general, you can choose the level of automation and maturity for your CD pipeline.

It's common to manage configuration files manually with pull requests (PRs). However, this makes it hard to increase deployment frequency, particularly at scale.

Another option is to create a CD solution using custom scripts and run them with general CI/CD tools like Jenkins or GitHub Actions. However, similar to environment promotion without Kubernetes, this becomes difficult to scale as you manage a mountain of custom scripts. 

Another option is to use a dedicated CD tool that works with Kubernetes and Kubernetes tools like Helm and Kustomize. Octopus Deploy is an example of a best-of-breed tool that helps automate environment promotion for Kubernetes or anything else.

## Conclusion

Environment promotion is critical to software delivery, but it gets challenging as you scale. Although many tools and solutions, like custom scripts, can automate environment promotion, they become difficult to manage when deploying hundreds or thousands of applications across multiple environments. A dedicated best-of-breed CD tool helps automate this process, whether you're deploying to Kubernetes or anywhere else. 

[Learn more about environment promotion using Octopus Deploy](use case page). 

Happy deployments!