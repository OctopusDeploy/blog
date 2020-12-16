---
title: Self-service runbooks for operations teams
description: Learn about self-service runbooks and how they benefit operations teams
author: adrian.howchin@octopus.com
visibility: public
published: 2020-12-07
metaImage: blogimage-self-service-runbooks-for-ops-teams-2020.png
bannerImage: blogimage-self-service-runbooks-for-ops-teams-2020.png
tags:
 - DevOps
 - Runbooks
---

![](blogimage-self-service-runbooks-for-ops-teams-2020.png)

In this post, I talk about runbooks from an operations team perspective, what they are, what we mean by self-service runbooks, and how they help you and your users.

## What are runbooks?

A runbook is simply a process that you follow to accomplish a task. You can have runbooks to restart a web server, clean up old log files, even swapping which environment a load balancer points to, among many others. 

Runbooks are a way to standardize work that typically happens often, ensuring the work gets done in a reliable, repeatable way. Because of the repeated nature of the work, runbooks are usually automated, allowing you to "bake into the code" the complex knowledge required to get the task done.

## What do we mean by self-service runbooks?

With a runbook, you are typically executing a task in response to a request from a user. For example, a user may raise a help ticket, asking you to refresh the data in their testing database. You have to spend the time reading the ticket, making sure you understand exactly what the user wants, and then execute the task. 

By making the runbook self-service, you are taking that same runbook you would use to refresh the test database and making it possible for the user who needs the refreshed test database to execute the task themselves. That user can now run that runbook whenever they need, without having to raise a ticket, and without taking up your valuable time.

## Why do self-service runbooks benefit operations teams?

By making runbooks available to your users, you save the time you would have spent performing the task. This frees you up for more important and interesting work. Additionally, because the user inputs all the required information directly into the runbook, you save time going back-and-forth on a ticket trying to gather all the correct information (and reducing the chance of mistakes in the process). 

You can also bake “guard-rails” into your self-service runbooks, ensuring that users can only go down the "safe path" when executing a runbook. For example, if you had a runbook that returned log files from production, you could hard-code which files are returned, ensuring the runbook cannot be abused to gain unauthorized access (and you don’t have to give out access to production). This can make life much easier when it comes to audit time, as you know the production system has only been accessed in a safe and compliant manner.

You can also create self-service runbooks that allow users to do very complex and time consuming tasks, such as creating a new AWS account. By doing this, your users have their needs met very quickly, and you can ensure every AWS account is created with all the correct VPC settings, password policy, and IAM roles for your users. 

## Self-service runbooks can help to reduce shadow IT

**Shadow IT** is the unapproved use of IT systems and applications within an organizations, and it is a big issue for many companies. Self-service runbooks can help to reduce shadow IT, as users are more likely to follow the right path when they can do so quickly and easily. 

> The goal of this step is controlled, self-service solutions. Any software you provide must meet two important criteria:
> - Self-service: Users must use the solution without bothering IT.
> - Control: IT must still be able to control data and user access.
> When you deliver controlled, self-service options, your business gets the best of both worlds. Users get the solutions they need quickly, and IT can still secure the data and applications."

[From MRC](https://www.mrc-productivity.com/blog/2016/07/6-ways-to-reduce-shadow-it-security-risks/)

By using self-service runbooks, users can solve their own problem without bothering IT, while operations teams can maintain control over the data and user access.

## Can a runbook help me during an outage?

Absolutely! By combining your monitoring and logging system with your runbooks, you can have your systems automatically trigger a runbook to respond to an outage. For example, your monitoring system may trigger a runbook when an HTTP server response time goes over a certain threshold, causing it to automatically gather logs and artifacts into a centralized place, then restart the web service.

## Conclusion

Not only do self-service runbooks save you time, but they also reduce the likelihood of errors, create happier users, and allow you to get on with doing more important and interesting work.

Happy deployments!
