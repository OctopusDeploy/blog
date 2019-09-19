---
title: "Octopus Cloud FAQs"
description: "Octopus Cloud: all your questions answered!"
author: andrew.katsivas@octopus.com
visibility: public
metaImage: metaimage-octopus-cloud.png
bannerImage: blogimage-octopus-cloud.png
published: 2018-05-30
tags:
 - Company
---

We've been talking about it for a while now, and Octopus Cloud is almost here - [welcome to the world of tomorrow](https://www.youtube.com/watch?v=aiwA0JrGfjA)! Since early in 2018 we've been in closed testing with a handful of users putting the platform through its paces, and in June 2018 we are opening it up to another group of early-access users, ahead of a public launch in Q3 2018. 

With that in mind, we've put together a few questions we get commonly asked, so you can get up-to-speed as quickly as possible. If you still have questions that we _haven't_ answered here, then please [contact our support team](mailto:support@octopus.com) and we'll be happy to answer them for you.

:::hint 
If you want occasional updates about our progress with Octopus Cloud, and to stay informed when we do officially publically launch, [register your interest](https://octopus.com/cloud/register-interest) and we'll keep you in the loop.
:::

## General

### When will Octopus Cloud be publically available?

We've been testing Octopus Cloud since Feb 2018, and an early-access period for selected users who registered their interest will be underway through June 2018. Therefore, we expect Octopus Cloud to be publically available in Q3 2018 (July-August).

### Can I import my existing deployment data from my self-hosted (on-premises) Octopus server into an Octopus Cloud instance?

We have plans to release migration tooling to help you seamlessly transition your existing self-hosted projects and configuration to a new cloud instance. However, in the early stages of release any projects will need to be set up from scratch, or [imported using `octo.exe`](https://octopus.com/docs/api-and-integration/octo.exe-command-line/import).

### I live somewhere that isn't near any of the available cloud regions: can I have my Octopus Cloud instance hosted nearer to my location?

At launch we will only be supporting a few specific cloud regions (initially Oregon, USA), but we have plans to bring more regions online as demand for instances grows.

If you want to specifically request a region, please send us an email at [support@octopus.com](mailto:support@octopus.com) and we'll add your request to the list.

## Features & Usage

### What is the difference between self-hosted (on-premises) Octopus and Octopus Cloud?

Octopus and Octopus Cloud are the same product, with the exception that Octopus Cloud is managed, monitored, backed up, and automatically upgraded for you by our team. However, some limited configuration and diagnostic functionality may not be available in Octopus Cloud (for security reasons).

Our end goal is that anything you can do in self-hosted Octopus, you should be able to do in the cloud-hosted system as well - this includes our exhaustive API and all steps and features you know and love.

### What if I want to cancel my Octopus instance: will I be able to export my data out?

Yes, of course! We will make sure you have the ability to retrieve a database backup of your Octopus instance before terminating your account.

### What can / can't I do with my Octopus Cloud instance? Are there limitations?

Generally speaking we've tried to empower our users as much as possible, without introducing any security or resource-sharing issues. We've written up our thoughts on the issue, and the sorts of things we would consider "excessive use" and what we would do about it, into an Acceptable Usage policy - you can [read the policy here](https://octopus.com/company/acceptable-usage).

## Security & Availability

### What uptime guarantees do you provide?

At launch we won't be able to provide any specific guarantees until we can be sure that we can meet them - and to do that, we need to have users on the platform so we can measure our availability. 

That being said, we will be doing our **absolute very best** to make sure you stay online and deploying with your Octopus Cloud, including having an engineer on-call 24/7 to action any incidents that get reported through our monitoring systems.

### Can Octopus staff access my data?

Octopus staff have no "standing-access" to your data: i.e., we can't just open up your account and rifle through your stuff. If you need assistance at any point, and you give us explicit permission to log in to your instance for the purposes of helping you, then we will do so: but only with your approval. In this event, any actions taken by an Octopus Support Engineer will be fully audited in your instance, so you can see exactly what we've done (if anything).

During other support events, such as restoring backups or responding to a request from you for encrypted data, we have the ability to recover your master key from encrypted storage and can use it to restore your data. In the case of recovering a failed instance, we may do this automatically. For other support scenarios, we may ask your explicit permission to access your master key.

### Who owns the data inside my Octopus instance?

All data inputted by you into your Octopus instance (including packages and scripts) are owned entirely by you: and, as such, if you cancel your Octopus license, that data is yours to take with you (see related question "What if I want to cancel my Octopus instance: will I be able to export my data out?").

We will, however, collect broad usage statistics (e.g., number of steps of a given type, deployment frequency etc.) so that we can understand customer usage patterns and tune the service to be as optimal as possible. These statistics are general in nature, and don't contain any specifics like project names, script contains and so forth.

### Do I need to take backups of my Octopus instance, or will you do it for me?

You do not need to take backups - that is all handled as a part of the hosted platform. We make regular backups of instance databases throughout the day and store these securely in blob storage, so we have multiple restore points in the unlikely case of disaster.

We'll be evaluating our backup and recovery strategy frequently after launch, to improve resiliency and recovery as we need to.

### Do I get full admin access to my instance? What is "Octopus Managers"

On your cloud instance there are a few select permissions that relate to the hosting of Octopus itself, and not so much the configuration/usability of your instance (e.g. server configuration logs), so we've introduced a new built-in team called "Octopus Managers": think of it as a "cloud-instance admin". The "Octopus Administrators" team is still present, but it's only used by our octoadmin account, and if you ask us to log in to your instance for support.

If you find there is something you think you ought to have access to as an "Octopus Manager", but don't, [let us know](mailto:support@octopus.com) and we can review the permissions.

## Technical

### Is the IP address of the Octopus Cloud instance static, so I can whitelist it on my network?

Unfortunately, not yet. The IP address is "sticky", in that it will be fixed while the instance is running - but if the instance gets restarted for any reason, a new IP address is likely to be assigned on reboot. Our infrastructure will automatically update your DNS record as part of this process, so it won't affect your regular use, but it may cause you pain with whitelisting. Our recommendation is that you use Tentacles in **polling mode** so you don't need your Octopus to reach into your network, until we have static IP support available. 

## Pricing

### Can I get a free trial of Octopus Cloud, to evaluate it before buying?

Of course! We offer a 30-day trial with no credit card required: just put in your details and we'll get the instance ready to go for you. About a week before the trial finishes we'll email you, to see how you're going and whether you want to continue using your cloud instance. If so, you will be able to select a pricing tier and set up payment information and seamlessly transition to a monthly plan. If not, there will be no hard feelings and we'll tear your instance down (with plenty of warning, don't worry!) and you won't have to pay anything at all.

### Can I upgrade my Octopus pricing tier once I've signed up?

Yes, you can scale your license up and down to suit your needs as often as it makes sense for you! Whenever you change tiers your limits will be adjusted straight away, and the pricing will change on the next billing cycle. [View Octopus Cloud pricing](https://octopus.com/cloud)

### If I cancel my Octopus Cloud instance mid-month, can I get a refund?

We bill Octopus Cloud one month at a time, so if you cancel you will be cancelling for the end of the paid period, at the end of the month. However, if you think you’re entitled to a refund, definitely let us know at [sales@octopus.com](mailto:sales@octopus.com) and we'll do our best to help.

### Can I pay for my Octopus Cloud instance annually?

Currently no, but we do intend to make this option available in the near future.

### Is there a free version of Octopus Cloud?

We'd absolutely **love** for this to be feasible, and are hoping to work towards it in the future, but at launch time there will be no free edition of Octopus Cloud, apart from the 30-day trial.  

### What if I need more nodes? Can I buy a HA / Data Center license for Octopus Cloud?

Not at initial launch, as there are some very unique challenges to be solved in the HA space. But it's definitely on our plan to implement in the near future.

---

This list of questions is certainly not exhaustive, so if you didn't find what you were looking for, please feel free to ask us! If it's something you think other users might benefit from knowing as well, put a question in the discussion section below; otherwise [reach out to us via email](mailto:support@octopus.com) and we'll do our best to assist.

Happy (cloud) deployments!