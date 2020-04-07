---
title: How we use Operations Runbooks for Octopus.com
description: How we're using runbooks at Octopus so that anybody on the team can restore Octopus.com if it ever goes down
author: Octopus
visibility: private
published: 3020-01-01
metaImage: 
bannerImage: 
tags:
  - DevOps
  - Runbooks
---

We recently shipped Operations Runbooks which help teams automate operations tasks like routine maintenance and emergency tasks. In this post, we look at how we use Runbooks to minimize downtime on Octopus.com and ensure anybody on the Octopus team (no matter their role at the company) can to respond to an unexpected outage and get the website back up and running.

## Octopus.com

Our website is mission critical. It allows our our customers to sign into Octopus Cloud, download the latest version of Octopus Server, read our blog, view our documentation, and it's how customers upgrade and manage their accounts. Most of the interactions our users have with us, is through the website.

The website typically gets hundreds of visitors an hour, so if Octopus.com goes down, it makes their life harder: They can't access their Octopus Cloud instance, the downloads they need, the documentation, support or anything else through the website. If somebody decides to try Octopus for the first time and they're met with the dreaded 404 page, our credibility is shot and they might never come back.

## How Octopus.com works

Octopus.com is powered by an ASP.NET Core web application that is hosted in Microsoft's Azure platform. We use Cloudflare to manage it's DNS and we use its load balancer to route traffic to the closest (geo-replicated) Azure app services (websites) available. If one region is unavailable (because there's an outage or during a deployment is underway), Cloudflare fails over to a alternate app service in one of the other Azure regions. It will redirect traffic to whichever web application is healthy. 

The **B** and **C** web apps are in different Azure regions for availability and speed, and we replicate the Azure SQL Server for database reads but database writes are sent to the primary database. 

![A diagram of the Cloudflare load balancer for Octopus.com](octopus-com.png)

TODO: Update diagram to use the v2 architecture.

## How do we manage an unexpected outage

If we have an unexpected outage, the first thing we do is let you, our customers, know that we're aware of the problem and that we're working on it.

TODO: Outline the overall process and mention the runbooks we use to help and why they're valuable. 

### Update status.octopus.com

We use [statuspage.io](https://www.statuspage.io/) to log incidents, which are then displayed on [status.octopus.com](https://status.octopus.com/) and sent out from our [Twitter account](https://twitter.com/OctopusDeploy).

### Runbook Example - Update IP Address Access Restrictions

Each Azure website has IP restrictions that only allow traffic from Cloudflare and the Octopus Deploy worker that does the deployment. This means we can’t access them directly, but Cloudflare can, which means we can use a runbook in Octopus to update the IP address, switching from the web apps that is currently down, to one that is up. 

![Octopus runbook to update the IP address](update-ip-step.png)

The advantage of this approach is that anybody at Octopus can run the runbook. They don't need to understand the script that is executed by the step, they don't personally need any of the permissions for the services involved or even a familiarity with those services.

They log into our Octopus instance and execute the runbook.

### Runbook Example - Failover database to secondary region

TODO: Write about why we do this. What it does etc. 



## Conclusion

By moving this process into an Operations Runbook with Octopus, our entire team can take steps to restore Octopus.com if it goes down, and better yet, it's all logged in Octopus and if we ever need to update the steps in the runbook with the certainty that if there's ever another incident, we don't have to worry about people following an outdated process.
