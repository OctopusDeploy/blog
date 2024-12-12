---
title: Why it’s critical to get environment promotion right
description: This blog post will explore environment promotion and why it can become challenging, particularly as you start to deploy to Kubernetes. 
author: nikita.dergilev@octopus.com
visibility: public
published: 2024-12-13
metaImage: blog-environment-promotion.png
bannerImage: blog-environment-promotion.png
bannerImageAlt: An image of an application being promoted across two environments
isFeatured: false
tags: 
  - DevOps
  - Kubernetes
---

Whether you are deploying to Kubernetes, or anywhere else, environment promotion is not a new concept, and most software teams will use some concept of it. However, it can get challenging to manage, particularly at scale. So what are those challenges, and how can you mitigate them?

This blog post will explore environment promotion and why it can become challenging, particularly as you start to deploy to Kubernetes. This blog post will cover:

- What environment promotion is and the typical structure
- Why it’s so important
- How environment promotion ties into Continuous Delivery 
- When it becomes challenging
- Solutions for getting environment promotion right
- What environment promotion looks like for Kubernetes


## What is environment promotion? 

Environment promotion, or environment progression, is a concept that isn’t new to software development. However, you can quickly face challenges as you scale if you haven’t set it up correctly. So let’s start with what it is. 

Environment promotion is the process of promoting an application or service across different environments, typically with increasing maturity. For example, you start by deploying to the development environment, then promote to staging for testing once you’ve ensured everything is working fine, and then afterward promote to production, where the intended users will use it. 

The concept of environment promotion is that every change happens via the promotion process with a combination of human approvals validating that code is ready for production or automation to run tests and deploy. Deployments must only happen via this promotion process, meaning no manual changes.  

## What does a typical environment structure look like? 

Each type of environment should have consistent configurations, limitations on what can be deployed there, and access policies. You can have whatever type of environment structure that works for you and your team, but typically most applications have three environment types:

- Development
- Pre-production
- Production

### Development environments 

Development environments let developers run and debug their code in environments outside their local development machine. Development environments can run branch versions of your application, and developers can usually access the infrastructure of these environments easily.

### Pre-production environments

Pre-production environments can be known by many names, including and test, but they all share a common purpose as the step between development and production. This environment is where you ensure that the application and deployment test will successful run in production, and so should mirror production environments. These environments should only run code from the main branch, to prevent changes that won't be pushed to your production environment. 

### Productions environments 

The most important environment is the production environment, as this is where your application serves customers. By this stage, code has been automatically tested and deployed multiple times to reduce the risk of something going wrong. 

## Why is the sequence in environment promotion so important? 

Having a structure in place for your environment promotion means that you can be confident in everything that has gone to production. If you have implemented strict rules for environment promotion, you know that what you have deployed to production will work as planned since the new version of your application and the pipeline itself (including any changes to it) have been tested in the pre-production environment.

Having confidence in what you’ve released is important for a few reasons:

- **Critical for testing deployments** - The only way to ensure that a deployment will work as expected is to test it. This is important not only for code and components but also for the deployment process. The best way to test the production deployment process is to use the same process in test environments. This lets you find and fix issues earlier, leading to a shortened feedback loop and less wasted time.
- **Reduced risk of something going wrong** - The more testing you do of new versions of an application before it goes to production, the less risk there is that something goes wrong that could create downtime. Environment promotion ensures that you have all the testing and gates to reduce the risk of something going wrong once it reaches customers. 
- **Crisis management** - Being confident in your deployment processes also helps during emergencies. Relying on a trusted, well-tested, automated process is much better than trying to improvise. Using an improvised process in a high-pressure situation makes you more likely to cause even more problems. 

## Automating environment promotion with Continuous Delivery

Environment promotion is not a new topic, and every software company has some form of environment and rules around promotion in place. Yet environment promotion is still a big challenge. 

One of the biggest benefits of Continuous Delivery (CD) and deployment automation is that it helps you increase deployment frequency, which helps you improve DORA metrics, as all focus on the quality of your environment promotion solution. 

If you don’t automate environment promotion, there is a higher risk of something going wrong in production because you haven’t tested it rigorously. This means it isn’t as easy to ship as frequently, so smaller changes accumulate, and a new release can introduce major changes to your application. This makes deployment time more uncertain, which usually needs to more testing to resolve potential conflicts of different features introduced between releases. You might need a dedicated Quality Assurance (QA) team and change management team to help with the process of approvals, which can make releases even more expensive, which can reduce release frequency further. 

However, if you have environment promotion in place, you’re more confident in what you ship, which means you can introduce more incremental changes. These changes are easy to test and have a smaller impact, so you can deliver value to your customers sooner and reduce the risk of something breaking. 

Environment promotion will help increase deployment frequency even more when you can automate the process, which is where CD comes in. CD provides an end-to-end automation of all the environments and tests. Ideally, as soon you commit new code, this code gets deployed to production in minutes with zero manual effort.

## When environment promotion becomes challenging

No matter which method you choose to automate your environment promotion, it should be pretty straightforward for the first few applications or microservices if you don’t have many environments or extra compliance or security requirements. However, it gets tricky as you start to scale, as you will likely scale in multiple directions simultaneously.  

First, the number of applications will grow. Depending on the size of your company, you can end up with hundreds, if not thousands, of individual applications you need to deploy independently. Some might have multiple updates daily, while others will get updates once every few months. Even if you reduce the number of updates to help manage the scale, this leads to the risk that the deployments won’t work if they’re not fully automated. 

The number of environments will also grow. For example, you might need to test multiple code versions simultaneously, so you’ll likely have many development environments. Eventually, you’ll need to provision them dynamically. You may also need to implement environments for specific customers, for early access or special QA environments to test your software in different integration scenarios.

Environments will also sometimes have more than one target, so you must consider how you deploy to a few targets under one environment, whether it is a Kubernetes cluster, a cloud service, a VM, or different geographic locations, branches, or clients. 

Finally, your pipeline will become increasingly complicated as you add sophisticated tests, notifications, and integrations to meet compliance and security requirements.

## How to address the challenges of environment promotion

So how can you automate environment promotion? There are a few options. You can write custom code to deploy your software to a given environment, run all the required checks, and then deploy to the next environment. Or you can use a dedicated tool to automate the process. 

Typically, if you use a generic tool like Jenkins or GitHub Actions that can be used for running any tasks, including Continuous Integrations and Continuous Delivery, you can expect to write more code, as these aren’t specialized. Using custom scripts can become very difficult to manage at this larger scale, and you’ll often end up with too many scripts to keep track of. This may cost you a huge extra cost of supporting a complex deployment landscape. 

If you use a dedicated CD tool, it will likely have a built-in model for environments, deployment steps, and lifecycles, without needing custom code. Your CD tool should be able to choose the right sequence and promotion conditions for each app, automating environment promotion for you at any scale. Examples of these tools would be Octopus Deploy or Codefresh. 

## Environment promotion for Kubernetes 

Unfortunately, Kubernetes doesn’t make environment promotion easy. The opposite is true. Kubernetes manifests combine cluster and app configuration, meaning you must manage both with one process. Not only that, but Kubernetes is a declarative platform. Therefore, it’s eventually consistent. After deployment, you need to know if the cluster achieved the desired state. So how do you solve environment promotion on Kubernetes? 

Conceptually, solving the environment promotion problem on Kubernetes isn’t different from environment promotion on any other platform. The difference in the tools we use: the extra capabilities and the limitations these tools introduce.
As mentioned above, Kubernetes is a declarative platform. The best way to deploy to Kubernetes is to create a configuration file describing your application and infrastructure parameters and apply it to a cluster. This approach is simple as configurations are easy to understand, and deployment is just one command. However, this is not ideal for environment promotions, as you must create and maintain a version of your configuration file per environment with every new deployment.

To solve these problems, the Kubernetes community introduced tools like Helm and Kustomize. With these tools, you can create a configuration once and alternate it with variables or simple patches per environment or tenant. These solutions are not as simple and human-readable as the plan YAML configuration, but it’s a good trade-off to enable configuration templating.

The Kubernetes community also introduced K8s-specific deployment tools like Argo CD and Flux CD. These tools continuously apply configurations (which could be plain YAML, Helm charts, or Kustomize manifests) from Git to a cluster, ensuring that a cluster always runs a configuration stored in Git. These tools follow a framework called GitOps.

A common approach to deployment using these tools involves organizing configuration files into folders, repositories, or branches. Each environment is represented by its configuration file(s), and a deployment tool synchronizes these files with the corresponding cluster.

The big unsolved problem here is the actual deployment automation. To deploy a new version of an application, we need to update a corresponding configuration file with the new app parameters (like a container image tag). We need to update these files according to the order of our environment promotion. Moreover, we still need to run tests, send notifications, perform policy checks, update change management tools, and take any other steps we would typically take to promote an application to an environment.

There are multiple ways to solve this problem for deployments to Kubernetes. As with deployments in general, you can choose the level of automation and maturity for your CD pipeline.

It is common to not automate the pipeline and manage configuration files manually with PRs. However, this makes increasing deployment frequency difficult, particularly at scale.

Another option is to create a CD solution using custom scripts and run them with general CI/CD tools like Jenkins or GitHub Actions. However, similar to environment promotion without Kubernetes, this becomes difficult to scale as you manage a mountain of custom scripts. 

Another option is to use a dedicated CD tool that works with Kubernetes and Kubernetes tools like Helm and Kustomize. Octopus Deploy is one example of a best-of-breed tool that helps automate environment promotion for Kubernetes or anything else. 

## Conclusion

Environment promotion is critical to software delivery, but it gets challenging as you scale. Although many tools and solutions, like custom scripts, can automate environment promotion, they become difficult to manage when deploying hundreds or thousands of applications across multiple environments. A dedicated best-of-breed CD tool helps automate this process, whether you are deploying to Kubernetes or anywhere else. 

[Learn more about environment promotion using Octopus Deploy](use case page) 

Happy deployments!