---
title: Why you should track vulnerabilities after deployment
description: Find out why you should track vulnerabilities after deployment, plus the ways how.
author: andrew.corrigan@octopus.com
visibility: private
published: 2022-09-06-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - DevOps
  - Containers
  - Cloud Orchestration
  - Testing
---

Dealing with vulnerabilities is an unfortunate but essential part of software development. There are [high-profile examples that highlight the importance of proactive risk management](https://octopus.com/blog/octopus-deploy-log4j-response), proving that responsibility for vulnerabilities doesn't stop at deployment.

In this post, I explore:

- Why you should track vulnerabilities after deploying your software
- Ways to track vulnerabilities and how to keep your users and business safe

## Vulnerabilities are found *after* deployment

It's a simple fact that software providers discover vulnerabilities *after* deployment, not before.

There are 2 semantic reasons for this:
 
- If testing picks up a would-be vulnerability before deployment, the code never makes it to an environment to be a vulnerability
- An undetected, yet-to-be deployed vulnerability isn't really a vulnerability until it hits an environment where someone can exploit it

Semantics aside, it's vital to know testing can't catch every problem in your code. Problems sneak through no matter how rigorous your internal testing, automated or otherwise, as you don't always know what to look for.

For this reason, you should never rely solely on internal pre-deployment testing. If you're not tracking vulnerabilities *after* deployment, then you're going to miss some.

## You want to be the ones finding a vulnerability

If you ship a vulnerability, it's best *you* find it before someone with bad intentions. Hackers thrive on unmonitored apps and infrastructure, and not checking for vulnerabilities makes their lives easier.

Failure to check for, find, and resolve vulnerabilities can lead to:

- Downtime
- Risks to company, employee, and customer data
- Damage to your company's reputation and earnings

You have a duty of care to your users to protect their interests. Not tracking vulnerabilities after deployment does your users a disservice, putting them and your own business interests at risk.

It's not just those with ill-intent you need to worry about, either. Often, security researchers and library authors find vulnerabilities in code dependencies long after they're first deployed. With a lot of time passing before discovery, it means there's a good chance others included the vulnerabilities in their own software and possibly iterated on it.

This brings us nicely to our next point.

## You don't want to accidentally iterate on a problem

The short sprints of DevOps processes help limit the likelihood of iterating on undiscovered problems. Short sprints also make it easier to pinpoint the update that introduced a problem.

If you don't track what happens after deployment, you reintroduce the risk of iterating on vulnerabilities. This can make vulnerabilities harder to find, troubleshoot, and fix before it's too late.

Checking for vulnerabilities after deployment can save yourself (and others) time and stress in the future.

## Managing vulnerabilities

Now you understand *why* you should track vulnerabilities after deployment, let's look at *how*.  
  
Here are a few proactive things you can do to help protect your interests.

### Patch and update your tools and infrastructure often

While you're responsible for the safety of your own product, providers of the tools and infrastructure you use are fallible too. Even the biggest software and service providers, such as Microsoft, need to fix exploits.

Amazon, as another example, uses the [Shared Responsibility Model](https://aws.amazon.com/compliance/shared-responsibility-model/). This model outlines expectations between the AWS services they offer and their customers. In a nutshell, AWS are responsible for security *of* the cloud, and customers are responsible for security *in* the cloud. This means you're responsible for the security of the operating systems and software you run on AWS infrastructure.

Regardless of your provider, though, you must ensure the products you use to deliver your software are:

- As up-to-date as your company's policies allow
- Patched and hot-fixed where urgent vulnerabilities exist
- Still compliant with the promises you make to your customers

You should regularly check service provider support and security pages for news on updates that will protect your product.

### Keep track of industry security news

While checking the support pages of your vendors, you should also keep tabs on the latest cyber security news.

The following websites report on new vulnerabilities and other cyber security concerns, alongside security management advice:

- [The Hacker News](https://thehackernews.com/)
- [Bleeping Computer](https://www.bleepingcomputer.com/)
- [Cybernews](https://cybernews.com/security/)
- [ZDNet](https://www.zdnet.com/topic/security/)

### Use a vulnerability scanner

You can also use vulnerability scanners, which proactively check code and tools for known vulnerabilities.

#### Example vulnerability scanners for code:

- [Snyk](https://snyk.io/)
- [GitHub Security](https://github.com/features/security)
- [Detectify](https://detectify.com/)
- [Checkov](https://www.checkov.io/)
- [CloudSploit](https://cloudsploit.com/cloudformation)

#### Example vulnerability scanners for infrastructure:

- [Intruder](https://www.intruder.io/)
- [ManageEngine Vulnerability Manager Plus](https://www.manageengine.com/vulnerability-management/integrated-vulnerability-and-patch-management.html)
- [Beagle](https://beaglesecurity.com/)
- [PortSwigger Burp Suite](https://portswigger.net/burp)  
- [Amazon Inspector](https://aws.amazon.com/inspector/)

## Conclusion

In this post, I explored why you should check for vulnerabilities after deployment, and looked at some strategies to help manage that.

Happy deployments!