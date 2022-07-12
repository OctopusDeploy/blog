---
title: Why you should track vulnerabilities after deployment
description: A surface-level look at why you should track vulnerabilities after deployment, plus the ways how.
author: andrew.corrigan@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: placeholderimg.png
bannerImage: placeholderimg.png
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - DevOps
  - Containers Series
  - Containers
  - Cloud Orchestration
  - Testing
  - Everything as Code
---

Dealing with vulnerabilities is an unfortunate but essential part of software development. Even recently, we’ve seen [high-profile examples that highlight the importance of proactive risk management](https://octopus.com/blog/octopus-deploy-log4j-response), and prove that responsibility for vulnerabilities doesn't stop at deployment.

In this post, we explore:

- Why you should track vulnerabilities after deploying your software
- A few ways to track vulnerabilities and how to keep your users and business safe

## All vulnerabilities are found *after* deployment

Let's zero in on the word 'after' from our page heading for a moment. It's a simple fact that software providers discover vulnerabilities *after* deployment, not before.

There are 2 semantic reasons for this:
 
- If testing picks up a would-be vulnerability before deployment, the code will never make it to an environment to be a vulnerability
- An undetected and yet-to-be deployed vulnerability is, instead, a *risk* until it hits an environment where people can exploit it

In either case, problems can sneak through and become vulnerabilities no matter how rigorous your internal testing, automated or otherwise.

For this reason, you should never rely solely on internal pre-deployment testing. If you're not tracking vulnerabilities *after* deployment, then you're not really tracking them at all.

## You definitely want to be the ones finding a vulnerability

If you ship a vulnerability, it's best that *you* find it before someone with bad intentions. Hackers thrive on unmonitored apps and infrastructure, and not checking for vulnerabilities makes their lives easier.

Failure to check for, find, and resolve vulnerabilities can lead to:

- Downtime
- Risks to company, employee, and customer data
- Damage to your company's reputation and earnings

You have a duty of care to your users, and doing everything you can to protect their interests is paramount. Not tracking vulnerabilities after deployment does your users a huge disservice, putting them and your own business interests at risk.

## You don't want to accidentally iterate on a problem

The short sprints of DevOps processes help limit the likelihood of iterating on undiscovered problems. Short sprints also make it easier to pinpoint the update that introduced a problem.

If you don't track what happens after deployment, you reintroduce the risk of iterating on vulnerabilities. This can make vulnerabilities harder to find, troubleshoot, and fix before it's too late.

Check for vulnerabilities after deployment and you can save yourself time and stress in the future.

## Managing vulnerabilities

Now we understand *why* you should track vulnerabilities after deployment, let's look at *how*.  
  
Here are a few proactive things you can do to help protect your interests.

### Patch and update your tools and infrastructure often

While you are responsible for the safety of your own product, providers of the tools and infrastructure you use are fallible too. Even the biggest software and service providers, such as Microsoft, need to fix exploits.

With that, it's important to ensure the products you use to deliver your software are:

- As up-to-date as your company's policies allow
- Patched and hot-fixed where urgent vulnerabilities exist
- Still compliant with the promises you make to your customers

You should also check service provider support and security pages for news on updates that will protect your product.

### Keep track of industry security news

While checking the support pages of your vendors, you should also keep tabs on the latest cyber security news.

The following websites report on new vulnerabilities and other cyber security concerns, alongside security management advice:

- [The Hacker News](https://thehackernews.com/)
- [Bleeping Computer](https://www.bleepingcomputer.com/)
- [Cybernews](https://cybernews.com/security/)
- [ZDNet](https://www.zdnet.com/topic/security/)

### Use a vulnerability scanner

Finally, you can use vulnerability scanners, which proactively check code and tools for known vulnerabilities.

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

## What's next?

In this post we explored why you should check for vulnerabilities after deployment, and looked at some strategies to help manage that.

Why not check out some of our recent Octopus blog posts, such as:

- link
- to
- some
- recent
- posts

Happy deployments (and follow-up testing)!