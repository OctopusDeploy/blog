---
title: What is shadow IT?
description: Shadow IT is the IT resources that an organization does not have visibility on. Find out how this affects your business!
author: terence.wong@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - DevOps
  - Runbooks Series
  - Runbooks
---

<!-- see https://github.com/OctopusDeploy/blog/blob/master/tags.txt for a comprehensive list of tags -->

## What is Shadow IT?

The IT department was once tasked with the oversight and management of all IT resources in an organization. This is no longer achievable because end users can create and use IT resources that the organization knows nothing about. This concept is known as shadow IT. 

Shadow IT is the use and management of IT technologies that are outside the control of an organization. In 2017, Gartner predicted that the IT department would make fewer technology decisions, and individual business units would begin to select technology for their teams, amounting to 38% of technology purchases[1]. In  2019, Everest Group predicted that upwards of 50% of technology spend in organizations is in the shadows[2]. With the rise of cloud solutions, this number is set to rise. 

How should IT departments respond to this? Is it realistic to track 100% of all IT purchases? Or should there be a more managed approach with accepted risk? What tools are they to help with this? This blog aims to explore these questions.

## Costs to the business

As more and more IT resources and infrastructure migrate to shadow IT, the risks of security breaches increase as they lie outside the control of IT departments. A study by EMC estimates that data loss and downtime contribute to $1.7 trillion each year due to shadow IT security breaches[3]. In IBM's 2021 Cost of Data Breach report, the average cost of a data breach rose from US$3.86 million to US$4.24million from 2020 to 2021[4].

![Average total cost of a data breach - IBM Cost of a Data Breach Report 2021](ibm.png "width=500")

There are also compliance concerns for businesses in highly regulated industries. Many companies, including the U.S army banned US soldiers from using TikTok over GDPR concerns[5]. For industries like defence or cybersecurity, compliance poses a management risk that shadow IT exacerbates. What cannot be tracked cannot be forced to comply.

Shadow IT affects operational costs. When shadow IT is left unmanaged, services become decentralized as each business unit is procuring IT for their own needs. One business unit may prefer one product while another prefers its competitor. This leads to underutilization of a prescribed centralized service. Shadow IT can also lead to unpredictible operation costs of cloud infrastructure. Think of all the unmonitored VM's that are spun up for a single purpose, are always running, but never get teared down. 

The costs of shadow IT boil down to a growing unknown resource that has operational and security risks.

## Why do employees use Shadow IT?

End users are any employees of an organization that require an IT resource to do their job. The main motivation of end users to employ shadow IT is convenience. IT policies can sometimes be rigorous. Often it is easier and faster to procure an IT solution themselves than go through the process with IT. End users could also prefer certain solutions over a prescribed solution which leads to more shadow IT. Rather than dealing with support tickets an end user may choose to find an alternate path. End users deviating from the prescribed IT solutions demonstrate the risks of shadow IT, however the end user is not often concerned with those risks. They want to get their job done in a way that is streamlined and efficient. At Octopus Deploy, we believe runbooks addresses the need for a streamlined experience, while maintaining the governance required to manage the IT resources.

[Image about the need for governance vs need to ease-of use and runbooks]


## Risk mitigation

The unknown nature of shadow IT increases the risk profile of an organization. Most likely, shadow IT has already invaded the majority of businesses, making the elimination of shadow IT entirely not possible. Gartner suggest three risk mitigation strategies to address this[6].

- Use Data Security Governance to Balance Local business unit IT (BUIT) Growth Objectives Against the Risk of Data Breaches and Financial Liabilities
- Deploy Shadow IT Discovery and Data Protection Tools to Enable the Safe Selection, Deployment and Notification of Unauthorized Cloud Services
- Use Data Security Governance to Develop and Orchestrate Consistent Security Policies Across All BUIT for Each Prioritized Dataset

From a business perspective, governance, discovery and protection are important to managing shadow IT. From an end user perspective, the IT solutions that are prescribed should be streamlined and minimise time spent in support. Self-service runbooks are able to address this.

## What do we mean by self-service runbooks?

A runbook is a way to execute a commonly repeated task. The runbook can be saved, reused or modified later by someone else. The typical runbook workflow for a user would be to raise a ticket with support who then spent time understanding and actioning the ticket to execute the desired task. The task could be to refresh the data in a test database.

A self-service runbook enables the user to execute the task themselves. Instead of waiting on support to refresh a data base, a self-service runbook with the required permissions can be supplied to the user who can run it whenever a database refresh is required. This can be extended to any number of use cases to tighten the feedback loop on operations tasks.

Imagine a self-service runbook for creating a new AWS account. These accounts have configurable parameters to set access levels, VPC settings, and other IAM considerations. If 50 different users were to try and set up an account, 50 slightly different types of users could be created. This immediately introduces shadow IT into the organization. If you apply this to creating VMs, container registries or other PaaS infrastructure it is easy to see how shadow IT grows.

Using self-service runbooks can restrict this process and make IT resources more standardized. This runbook can be configured to set the resource up with the appropriate access and monitoring controls required to manage the resource.

Through not an exhausive solution, runbooks are able to improve governance of IT resources and improve ease-of-use for end users.

> The goal of this step is controlled, self-service solutions. Any software you provide must meet two important criteria:
> - Self-service: Users must use the solution without bothering IT.
> - Control: IT must still be able to control data and user access.
> When you deliver controlled, self-service options, your business gets the best of both worlds. Users get the solutions they need quickly, and IT can still secure the data and applications."

[From MRC](https://www.mrc-productivity.com/blog/2016/07/6-ways-to-reduce-shadow-it-security-risks/)

Self-service runbooks allows operations teams to ensure standardization into the processes that end users are running. Streamlining a process from multiple different ways into one standardized way that works repeatedly can provide a big benefit to a business.

## Conclusion

Shadow IT is any IT resource that lies outside the control of the organization. It is a problem with several risks and high costs to businesses. To manage shadow IT, busineses need more governance, discovery and protection of IT assets. End users want more streamlined processes and the ability to solve problems without too many support tickets. Runbooks are a way to solve this issue by providing a self-serviced way to run commonly used tasks. Applying this concept to a problem like setting up cloud accounts provides standardization for IT assets

## Learn more

- [Gartner 38%](https://www.gartner.com/smarterwithgartner/make-the-best-of-shadow-it)
- [Everest Group 50%](https://www.everestgrp.com/2019-04-why-shadow-it-is-the-next-looming-cybersecurity-threat-in-the-news-49881.html/)
- [EMC](https://corporate.delltechnologies.com/en-us/newsroom/announcements/2014/12/20141202-01.htm)
- [IBM](https://www.ibm.com/au-en/security/data-breach)
- [TikTok](https://www.forbes.com/sites/carlieporterfield/2020/01/02/us-army-bans-soldiers-from-using-tiktok/?sh=5bb4b66deb9b)
- [Risk management](https://www.gartner.com/smarterwithgartner/make-the-best-of-shadow-it)

!include <q2-2022-newsletter-cta>

Happy deployments! 