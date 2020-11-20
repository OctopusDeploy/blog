---
title: Self-service runbooks for Operations Teams
description: Learn about self-service runbooks and how they benefit operations teams
author: adrian.howchin@octopus.com
visibility: public
published: 2020-12-24
metaImage: blogimage-integrating-octopus-and-grafana.png
bannerImage: blogimage-integrating-octopus-and-grafana.png
tags:
 - Product
---

![Self-service runbooks for Operations Teams](blogimage-integrating-octopus-and-grafana.png)

In this post, we will look from an Operations Team perspective at what a runbook is, what we mean by self-service runbooks, why you would want to use self-service runbooks, and how they help both you and your users.

## What are runbooks?

A runbook is simply a process that you follow for a task. You can have runbooks for almost anything - restarting a web server, cleaning up old log files, or even swapping which environment a load balancer points to.They are a way to standardize work that typically happens often, ensuring that the work gets done the same way each time.

## What do we mean by self-service runbooks?

With a runbook, you are typically executing a task in response to a request from a user. For example, a user may raise a help ticket with you, asking you to refresh the data in their testing database. You have to spend the time reading the ticket, making sure you understand exactly what the user wants, and then execute the task. 

By making the runbook self-service, we mean that you are taking that same runbook you would use to refresh the test database, and making it available to the user who wants to run it. The user can now run that runbook whenever they like, without needing to raise a ticket, and without taking up your valuable time.

## How do self-service runbooks benefit Operations Teams?

By making runbooks available to your users, you save the time you would have spent performing the task, freeing you up for more valuable and interesting work. Additionally, because the user inputs all the required information directly into the runbook, you save time going back-and-forth on a ticket trying to gather all the correct information (and reducing the chance of mistakes in the process). 

You can also bake “guard-rails” into your self-service runbooks, ensuring that users only go down the correct path when executing a runbook. For example, if you had a runbook that returned log files from production, you could hard-code which files are returned, ensuring that the runbook cannot be abused to gain unauthorised access (and that you don’t have to give out access to production!). This can make life much easier when it comes to audit time, as you know that the production system has only been accessed in a safe and compliant manner.

You can also create self-service runbooks that allow users to do very complex and time consuming tasks, such as creating a new AWS account. By doing this, your users have their needs met very quickly, and you can rest easy knowing that the AWS account was created with all the correct VPC settings, password policy, and IAM roles for your users. 

Finally, self-service runbooks also help reduce shadow IT, as users are more likely to follow the right path when they can do so quickly and easily.

## Can a runbook help me during an outage?

Absolutely! By combining your monitoring and logging system with your runbooks, you can have your systems automatically trigger a runbook to respond to an outage. For example, your monitoring system may trigger a runbook when a HTTP server response time goes over a certain threshold, causing it to automatically gather logs and artifacts into a centralised place, then restarting the web service.

## Conclusion

Not only do self-service runbooks save you time, but they also reduce the likelihood of errors, create happier users, and allow you to get on with doing more important and interesting work.

Happy deployments!