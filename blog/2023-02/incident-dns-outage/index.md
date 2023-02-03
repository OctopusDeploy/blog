---
title: "Outage on octopus.com - report and learnings"
description: Public incident report and our learnings about the octopus.com DNS disruption from January 25 – 26, 2023.
author: alix.klingenberg@octopus.com
visibility: public
published: 2023-02-03
metaImage: blogimage-incidentdnsoutage-2023x2.png
bannerImage: blogimage-incidentdnsoutage-2023x2.png
bannerImageAlt: A stylized Octopus and shield icon
isFeatured: false
tags:
  - Engineering
  - Incident Reports
---

From 3:37pm UTC Wednesday, January 25, 2023 to 3:44pm UTC Thursday, January 26, 2023, customers in some regions couldn’t access octopus.com/signin, the blog, the docs, and other URLs, as they returned HTTP 404 (page not found) responses. This mostly affected the US and EU, with some intermittent impact in regions in Asia.

In this post, we detail what happened, our response, and our learnings.

## How the incident started

On January 25, we undertook scheduled maintenance on the `octopus.com` and `octopusdeploy.com` domains as part of a planned DNS and routing provider migration. In this step of the migration, the routing setup in one Azure Front Door (AFD) profiles was transferred to another AFD profile. The new AFD profile didn’t behave as expected.

We identified that a subset of requests in certain regions were incorrectly routed by the new AFD profile, resulting in HTTP 404 responses. We publicized the service interruption for the scheduled maintenance on our Status Page. 

We treated this as an incident because the maintenance didn’t go as planned and there were complicating factors:

- A network outage from Azure
- An internal routing issue in AFD
- The new WAF created significant noise in our applications
- The issue occurred during an Australian public holiday where many of our engineers are based

The changes to the AFD profiles couldn’t be safely rolled back.


## Key timings

| Event                        | Time period  |
| ---------------------------- | ------------ |
| Time to detection            | 10hrs 21mins |
| Time to incident declaration | 13hrs 27mins |
| Time to resolution           | 37hrs 44mins |

## Incident timeline

_(All dates and times below are shown in UTC)_

### Wednesday, January 25, 2023

**_02:00_** An Octopus engineer deployed a change to the `octopus.com` and `octopusdeploy.com` domains, moving the associated routing between Azure Front Door profiles as part of a migration project.

**_02:39_** The engineer switched the AFD WAF from prevention mode to detection mode due to it blocking valid traffic.

**_07:05 _** A network outage in Azure obscured the issue - see our [Octopus Cloud connectivity disruption report](https://octopus.com/blog/cloud-connectivity-disruption-report-learnings).

**_12:21_** An application support engineer in the EU region reported experiencing intermittent 404s on the `octopus.com` domain.

**_12:43_** Azure’s network outage was fully resolved.

**_14:06_** First customer-facing issue was reported.

**_14:06 - 15:27_** Application support engineers investigated the issue and determined it was intermittent and affecting customers in EU and US regions.

:::hint
**_15:27_** Status Page updated: An incident was declared.
:::

**_15:27 - 17:06_** Investigations showed that the web app and database for the affected routes weren't under any significant load. The new AFD profile reported a warning on the `octopus.com` CNAME record.

**_17:06_** The on-call engineer determined the issue was in AFD as an endpoint dynamically resolved to 2 IP addresses, and one IP address consistently returned a 404.

:::hint
**_17:22_** Status Page updated: We are continuing to investigate this issue.
:::

**_17:35_** Azure support ticket raised with "Severity: B".

:::hint
**_18:32_** Status Page updated: Identified affected regions.
:::

**_20:32_** The incident was escalated to the engineer responsible for the AFD migration project.

**_20:32 - 21:42_** We investigated the warning about CNAME records on the 2 domains in AFD.

:::hint
**_21:36_** Status Page updated: Internal escalation.
:::

**_21:42_** We had a call with an Azure support rep, who advised the warning about CNAME records was likely the root issue.

**_21:42 - 22:26_** Attempted mitigation steps based on assumption that CNAME flattening was the root cause.

**_22:26_** CNAME flattening was ruled out as the cause based on successful requests to one of the 2 IP addresses resolved from the affected endpoint. It had also successfully worked on the previous AFD profile for approximately a year.
Our engineers focused on the failing route in the AFD profile.

**_22:37_** The support ticket with Azure was escalated to "Severity: A".

**_22:37 - 02:01_** Our engineers worked closely with the Azure support reps to provide enough information for Azure to identify and reproduce the issue.

:::hint
**_22:59_** Status Page updated: Cloud provider escalation.
:::

### Thursday, January 26, 2023

**_02:01_** A new potential root cause was identified, leading to a support request being raised with original DNS provider.

**_02:22_** An Azure application support engineer noted a high volume of 404 requests across the AFD profile, due to having set the mode of the WAF to `detection` instead of `prevention`.

**_03:12_** The incident was escalated internally to bring in senior engineering management staff due to the length of time without a clear path to resolution.

:::hint
**_04:15_** A problem with the routing was identified: renewed focus on the AFD profile.
:::

**_04:19_** Status Page updated: Incident status changed to identified.

**_05:42_** Previous cloud provider advised that their hosts were resolving to the correct IP addresses and recommended requesting public resolvers to flush their DNS caches.

**_05:43_** Potential mitigation of using `www` subdomain to control request resolution in AFD suggested.

**_07:06_** Azure advised that AFD could not support CNAME flattening.

**_07:13_** Internal escalation to involve Trust & Safety and SecOps.

**_07:31_** Azure confirmed the only path to resolution they saw was moving all DNS records to Azure DNS. Azure confirmed they'd sync the correct configuration to all their edge nodes.

**_08:49_** We created a new AFD endpoint for `www.octopus.com` and added a redirect from the CNAME with the existing DNS provider.

**_08:53_** Updated app services to accept the new origin header.

**_09:26_** Disabled app-level redirect from `www.octopus.com` to `octopus.com`.

**_10:05_** Updated config for affected SSO services.

**_10:07_** Changed the root octopus.com to be a CNAME and created a 302 redirect from `octopus.com` to `www.octopus.com`.

**_10:18_** Testing showed previously broken routes working again.

:::hint
**_11:07_** Status Page updated: Incident status changed to `Monitoring` and advice issued on how to resolve.
:::

**_11:10_** Internal reports showed that affected EU regions flipped from “mostly not working” to “mostly working”.

**_11:49_** Internal reports showed that `octopus.com` was still 404ing.

**_12:19_** Confirmed that the routing issue inside AFD was still happening despite a catch-all rule that should have been applied.

**_12:36_** Attempted to force AFD to route certain URLs by creating an explicit set of rules.

:::hint
**_12:38_** Status Page updated: Incident status changed back to `identified`.
:::

**_13:23_** Azure confirmed they had reproduced the issue and engaged a Product Group for a fix.

**_14:30_** Availability tests started reporting success.

**_14:53_** Azure confirmed that a fix was applied an hour ago.

:::hint
**_15:07_** Status Page update: Incident status changed back to monitoring.
:::

:::hint
**_15:44_** Status Page update: Incident resolved.
:::

## Technical details

The 404s were occurring because customers' requests were being routed to the incorrect web app. The 404 responses were region specific and intermittent. Working out the exact cause of the incorrect routing was complex. We needed to dive into the relationship between the DNS provider and the endpoint routing in Azure Front Door (AFD).

### What we observed

A specific AFD endpoint, `octo-public-prod-endpoint-f2e5gmecbdf3bfg9.z01.azurefd.net`, for the `octopus.com` domain was dynamically resolving to one of 2 IP addresses. One of the 2 IP addresses returned a 404 on all the associated routes, `/signin`, `/docs`, and `/blog`, while the other IP address worked. 

This indicated the CNAME warning was unlikely the cause, as some traffic was being successfully routed.

We discovered the 2 IP addresses that the AFD route resolved to were different from the IP addresses of the A-B pair of the web apps expecting to receive this traffic.

### What happened

AFD doesn’t update all its nodes when a routing change happens, so some AFD nodes still directed traffic to the incorrect web application. The affected endpoint was resolving to the incorrect application before the migration. Without control of a CNAME record, AFD couldn’t authoritatively update the flattened A records.

We set up a CNAME record on AFD to give them authoritative control of all requests on that subdomain. The incorrect IP address resolution behavior on the affected endpoint continued.

The incorrect routing inside AFD stopped after Azure advised us they’d made internal changes.

## Remediation

There were 2 red herrings that weren’t contributing factors:

- The 404 errors from the WAF
- The CNAME flattening on the apex domain

In practice, removing the proxy and switching to AFD’s WAF is irrelevant to this issue - we included them here because they created a red herring when attempting to determine the causes.

Similarly, using CNAME flattening on the apex domain made the causes unclear. CNAME flattening is where the DNS provider creates a series of A Records instead of a single CNAME record for dynamic resolution of the underlying IP address. You can learn more in this [explainer by Cloudflare](https://blog.cloudflare.com/introducing-cname-flattening-rfc-compliant-cnames-at-a-domains-root/).

Based on our best understanding of the problem, we created a `www` subdomain on the affected domains. We created a CNAME record on this subdomain to give AFD authoritative control over resolving all requests to that domain. We then redirected traffic from the affected domains to the appropriate `www` subdomains.

This mitigation helped initially, but the incorrect IP address resolution in the AFD endpoint reappeared. The IP issue resolved after Azure advised they'd made changes to our AFD profile. 

We understand Azure moved the affected AFD profile “from a legacy platform to the most updated platform,” but we haven’t had official confirmation at the time of writing.

We considered rolling back the AFD profile changes after we understood the problem. We decided against this because we knew only a subset of the AFD nodes would update, leaving some regions in a broken state.

## Next steps

To help us understand this incident in more depth, we've engaged with our Azure account manager on this issue.

Immediately following resolution, we addressed the impact of the `www` subdomain mitigation, particularly around CORS, static assets, and SEO.

After we were back to a stable state, we continued with the migration project. We adapted the project to work closely with Azure to ensure that our changes work as expected. We continue to have scheduled downtime as we progress our migration. Based on our learnings, we can design rollback-safe migration steps. This helps us minimize the impact of scheduled downtime by rolling back immediately when there's unexpected behavior.

We ran an incident review and identified areas of improvement:

- Development process: We are reviewing the quality controls in our development process to identify where we could have detected the problem before applying the changes to our production environment.

- Incident response process for long-running incidents: We haven't experienced an incident over this length of time before. We started to run out of rested on-call engineers with appropriate context to manage the incident. We're investigating how other internal and external groups manage these situations and are updating our on-call rosters and guidance accordingly.

## Conclusion

Octopus Deploy takes service availability seriously. The time to resolution on this incident was longer than we aim for. We apologize for the disruption to our customers due to this outage.

We're continuing to investigate the complexities of this incident so our remediations will be effective and our customers can focus on happy deployments.