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

In a traditional organization, the IT department oversees and manages all IT resources, however, with such easy access to cloud-based IT resources, many employees (who are impatient and have pressing deadlines) find it easier and quicker to spin up IT infrastructure themselves, rather than filling out request and waiting for the IT department to fullfil the request. When employees create and use their own IT resourcers that are invisible to the IT department, this is known as shadow IT.

In 2017, Gartner predicted that the IT department would make fewer technology decisions, and individual business units would begin to select technology for their teams, amounting to 38% of technology purchases[1]. In 2019, Everest Group predicted that more than 50% of technology spending in organizations was due to shadow IT[2]. The rise of cloud technology compounds this problem, making it easier than ever for employees to use unapproved IT resources.

Shadow IT poses a number of new questions for oragnizations, for instance:

- How should IT departments respond to Shadow IT? 
- Is it realistic, or even practical, to track 100% of all IT resources?
- Should there be a more managed approach with accepted risk?
- What tools can help manage Shadow IT? 

This blog aims to explore these questions.

## Costs to the business

More and more teams are taking advantage of shadow IT. This increases the risks of security breaches as the resouces are outside the control of IT department. A study by EMC estimates that data loss and downtime contribute to $1.7 trillion each year due to shadow IT security breaches[3]. In IBM's 2021 [Cost of Data Breach Report](https://www.ibm.com/au-en/security/data-breach), the average cost of a data breach rose from US$3.86 million to US$4.24million from 2020 to 2021[4].

![Average total cost of a data breach - IBM Cost of a Data Breach Report 2021](ibm.png "width=500")

There are also compliance concerns for businesses in highly regulated industries. Many organizations, including the U.S Army, banned US soldiers from using TikTok over GDPR concerns[5]. Compliance poses a management risk that shadow IT exacerbates. Companies cannot ensure compliance on what they cannot track.

Shadow IT affects operational costs. When shadow IT is left unmanaged, services become decentralized as each business unit procures IT for its own needs. One business unit may prefer one product while another prefers its competitor. Shadow IT can also lead to unpredictable operation costs of cloud infrastructure. Think of all the unmonitored VM's created for a single purpose, always running but never getting torn down. By allowing business units to procure their own IT infrastructure, businesses lose then benefits of their buying power and abilty to reduce the cost of IT infrastructure.

The true costs of shadow IT come down to a growing unknown resource that has operational and security risks.

## Why do employees use shadow IT?

End users are any employees of an organization that require an IT resource to do their job. The primary motivation of end-users to employ shadow IT is convenience. IT policies can sometimes be rigorous. Often it is easier and faster for the end-user to procure an IT solution themselves than go through the process with IT. End users often also prefer specific solutions over a prescribed solution, leading to more shadow IT. Rather than dealing with support tickets, an end-user may find an alternate solution to the problem that introduces shadow IT. 

End users are the cause of shadow IT, but they are generally not concerned with the consequences. They want to get their job done in a streamlined and efficient way. Self-service runbooks can address this by ensuring a streamlined experience with governance that gives the end-user the ability to spin up the infrastructure they need without avoiding the IT department.

[Image about the need for governance vs need to ease-of use and runbooks]


## Risk mitigation

The unknown nature of shadow IT increases the risk profile of an organization. Shadow IT has already infected businesses and will only grow. It is a matter of managing the risk. Gartner suggests three [risk mitigation strategies]() to address this[6].

> - Use Data Security Governance to Balance Local business unit IT (BUIT) Growth Objectives Against the Risk of Data Breaches and Financial Liabilities
> - Deploy Shadow IT Discovery and Data Protection Tools to Enable the Safe Selection, Deployment, and Notification of Unauthorized Cloud Services
> - Use Data Security Governance to Develop and Orchestrate Consistent Security Policies Across All BUIT for Each Prioritized Dataset

Shadow IT requires governance, discovery, and protection. The solution must be streamlined and minimize time spent in support. Self-service runbooks can address this.

## What is a self-service runbook?

A runbook is a reusable way to execute a commonly repeated task. The task could be to refresh the data in a test database. A user raises a ticket with support, who then spends time understanding and actioning the ticket to perform the desired task. 

A self-service runbook enables the user to execute the task themselves. A user can now run the runbook to refresh the database without support. Self-service runbooks can apply to other use cases to tighten the feedback loop on operations tasks.

Imagine a self-service runbook for creating a new AWS account. Users need to set access levels, VPC settings, and other IAM considerations. If 50 different users were to try and set up an account, this could result in 50 different types of users. Unmanaged creation of user accounts introduces shadow IT into the organization. If you apply this to creating VMs, container registries or other PaaS infrastructure, it is easy to see how shadow IT grows.

Using self-service runbooks can restrict this process and make IT resources more standardized. Operations can use the runbook to enable monitoring and security on the IT resource

Though runbooks don't solve every aspect of shadow IT, runbooks can improve IT resource governance and ease of use for end-users. According to MRC on managing shadow IT risk[7]:

> The goal of this step is controlled, self-service solutions. Any software you provide must meet two important criteria:
> - Self-service: Users must use the solution without bothering IT.
> - Control: IT must still be able to control data and user access.
> When you deliver controlled, self-service options, your business gets the best of both worlds. Users get the solutions they need quickly, and IT can still secure the data and applications."


Self-service runbooks allow operations teams to enable monitoring, provide security. They enable the user to self-serve their problems without support.

## Conclusion

Shadow IT is any IT resource that lies outside the organization's control. It is a problem with several risks and high costs to businesses. Businesses need more governance, discovery, and protection of IT assets. End users want more streamlined processes and solve problems without too many support tickets. Runbooks can solve this issue by providing a self-service way to run commonly used tasks. Applying this concept to a problem like setting up cloud accounts provides standardization for IT assets

## Learn more

- [Gartner 38%](https://www.gartner.com/smarterwithgartner/make-the-best-of-shadow-it)
- [Everest Group 50%](https://www.everestgrp.com/2019-04-why-shadow-it-is-the-next-looming-cybersecurity-threat-in-the-news-49881.html/)
- [EMC](https://corporate.delltechnologies.com/en-us/newsroom/announcements/2014/12/20141202-01.htm)
- [IBM](https://www.ibm.com/au-en/security/data-breach)
- [TikTok](https://www.forbes.com/sites/carlieporterfield/2020/01/02/us-army-bans-soldiers-from-using-tiktok/?sh=5bb4b66deb9b)
- [Risk management](https://www.gartner.com/smarterwithgartner/make-the-best-of-shadow-it)
- [MRC](https://www.mrc-productivity.com/blog/2016/07/6-ways-to-reduce-shadow-it-security-risks/)

!include <q2-2022-newsletter-cta>

Happy deployments! 
