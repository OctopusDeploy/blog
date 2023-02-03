---
title: "Octopus Cloud, blogs and docs 404s - report and learnings"
description: Public incident report and our learnings about the octopus.com DNS disruption from January 25-26, 2023.
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

Wednesday, 25 January, 2023 - Thursday, 26 January, 2023

From 15:37 Wednesday, 25 January, 2023 to 15:44 Thursday, 26 January, 2023 UTC, customers in some regions experienced intermittent HTTP 404 responses to requests to octopus.com/signin, /blog, /docs and other URLs. This prevented customers from signing in to their Octopus Cloud instances and accessing any Octopus blogs or docs. The affected regions were primarily US and EU, with some additional intermittent impact in some regions in Asia.

On 25 January we scheduled maintenance on the `octopus.com` and `octopusdeploy.com` domains as part of a planned DNS and routing provider migration. In this step of the migration, the routing setup in one Azure Front Door (AFD) profile was transferred to another AFD profile. The new AFD profile did not behave as expected.

We identified that a subset of requests in certain regions were being incorrectly routed by the new AFD profile, resulting in HTTP 404 responses. The service interruption for the scheduled maintenance was publicised on our Status Page. We treated this as an incident because the maintenance didn’t go as planned and was complicated by a number of factors:

- A network outage from Azure
- An internal routing issue within AFD
- The new WAF created significant noise in our applications
- The issue occurred during an Australian public holiday.

The changes to the AFD profiles could not be safely rolled back.

## Key timings

| Event                        | Time period  |
| ---------------------------- | ------------ |
| Time to detection            | 10hrs 21mins |
| Time to incident declaration | 13hrs 27mins |
| Time to resolution           | 37hrs 44mins |

## Incident timeline

_(All dates and times below are shown in UTC)_

### Wednesday, January 25, 2023

:::hint
**_02:00_** An Octopus engineer deployed a change to the `octopus.com` and `octopusdeploy.com` domains, moving the associated routing between Azure Front Door profiles as part of a migration project.
:::

**_02:39_** The engineer switched the AFD WAF from prevention mode to detection mode due to it blocking valid traffic.

**_07:05 _** A network outage in Azure obscured the issue - see our [incident report](blog/2023-02/incident-dns-outage/index.md).

**_12:21_** An application support engineer in the EU region reported that they were experiencing intermittent 404s on the `octopus.com` domain.

**_12:43_** Azure’s network outage is fully resolved.

**_14:06_** First customer-facing issue was reported.

**_14:06 - 15:27_** Application support engineers investigated the issue and determined that it was intermittent and affecting customers in EU and US regions.

:::hint
**_15:27_** Status Page update: An incident was declared.
:::

**_15:27 - 17:06_** Investigation shows that the web app and database for the affected routes are not under any significant load. The new AFD profile is reporting a warning on the `octopus.com` CNAME record.

**_17:06_** The on-call engineer determines that the issue lies within AFD as an endpoint dynamically resolves to two IP addresses, and one IP address consistently returns a 404.

:::hint
**_17:22_** Status Page update: We are continuing to investigate this issue
:::

**_17:35_** Azure support ticket raised with Severity: B.

:::hint
**_18:32_** Status Page update: Identified affected regions
:::

**_20:32_** The incident is escalated to the engineer responsible for the AFD migration project.

**_20:32 - 21:42_** Investigated the warning about CNAME records on the two domains in AFD.

:::hint
**_21:36_** Status Page update: Internal escalation
:::

**_21:42_** Call with Azure support rep, who advised the warning about CNAME records is likely the root issue.

**_21:42 - 22:26_** Attempted mitigation steps based on assumption that CNAME flattening was the root cause.

**_22:26_** CNAME flattening was ruled out as the cause based on successful requests to one of the two IP addresses resolved from the affected endpoint. It had also successfully worked on the previous AFD profile for approx. a year.
Engineers focused on the failing route in the AFD profile.

**_22:37_** The support ticket with Azure was escalated to Severity: A.

**_22:37 - 02:01_** Engineers worked closely with the Azure support reps to provide enough information for Azure to identify and reproduce the issue.

:::hint
**_22:59_** Status Page update: Cloud provider escalation
:::

### Thursday, January 26, 2023

**_02:01_** New potential root cause identified leading to support request raised with original DNS provider.

**_02:22_** Azure application support engineer noted a high volume of 404 requests across the AFD profile, due to having set the mode of the WAF to `detection` instead of `prevention`.

**_03:12_** Incident was escalated internally to bring in senior engineering management staff due to length of time passed without a clear path to resolution.

:::hint
**_04:15_** Problem with the routing was identified: renewed focus on the AFD profile.
:::

**_04:19_** Status Page update: Incident status changed to identified.

**_05:42_** Previous cloud provider advised that their hosts were resolving to the correct IP addresses and recommended requesting public resolvers to flush their DNS caches.

**_05:43_** Potential mitigation of using www subdomain to control request resolution in AFD suggested.

**_07:06_** Azure advised that AFD could not support CNAME flattening.

**_07:13_** Internal escalation to involve Trust & Safety and Secops.

**_07:31_** Azure confirmed that the only path to resolution they saw was to move all DNS records to Azure DNS. Azure confirmed that they would sync the correct configuration to all their edge nodes.

**_08:49_** Created a new AFD endpoint for `www.octopus.com` and added a redirect from the CNAME with the existing DNS provider.

**_08:53_** Updated app services to accept the new origin header.

**_09:26_** Disabled app-level redirect from `www.octopus.com` to `octopus.com`.

**_10:05_** Updated config for affected SSO services.

**_10:07_** Changed the root octopus.com to be a CNAME and created a 302 redirect from octopus.com to www.octopus.com.

**_10:18_** Testing shows previously broken routes working again.

:::hint
**_11:07_** Status Page update: Incident status changed to `monitoring` and advice issued on how to resolve.
:::

**_11:10_** Internal reports that affected EU regions had flipped from “mostly not working” to “mostly working”.

**_11:49_** Internal reports that `octopus.com` was still 404ing.

**_12:19_** Confirmed that the routing issue inside AFD was still happening despite a catch-all rule that should have been applied.

**_12:36_** Attempted to force AFD to route certain URLs by creating an explicit set of rules.

:::hint
**_12:38_** Status Page update: Incident status changed back to `identified`.
:::

**_13:23_** Azure confirmed that they had reproduced the issue and engaged a Product Group for a fix.

**_14:30_** Availability tests started reporting success.

**_14:53_** Azure confirmed that a fix was applied an hour ago.

:::hint
**_15:07_** Status Page update: Incident status changed back to monitoring
:::

:::hint
**_15:44_** Status Page update: Incident resolved
:::

## Technical details

The 404s that were observed by customers were due to their requests being routed to the incorrect web app. The 404 responses were region specific and intermittent. Pinning down the exact cause of the incorrect routing is a complex operation, requiring us to dive into the relationship between the DNS provider and the endpoint routing in Azure Front Door (AFD).

### What we observed

A specific AFD endpoint - `octo-public-prod-endpoint-f2e5gmecbdf3bfg9.z01.azurefd.net` - for the `octopus.com` domain was dynamically resolving to one of two IP addresses. One of the two IP addresses 404ed on all the associated routes - /signin, /docs, /blog, while the other IP address worked. This indicated that the CNAME warning was unlikely to be the cause, as some traffic was being successfully routed.
We discovered that the two IP addresses that the AFD route resolved to were different from the actual IP addresses of the A-B pair of the web-apps that were expecting to receive this traffic.

### What happened

AFD does not update all of its nodes when a routing change happens, so some AFD nodes were still directing traffic to the incorrect web application. The incorrect application was what the affected endpoint was resolving to before the migration. Without control of a CNAME record, AFD was unable to authoritatively update the flattened A records.
We set up a CNAME record on AFD in order to give them authoritative control of all requests on that subdomain. The incorrect IP address resolution behaviour on the affected endpoint continued.
The incorrect routing inside AFD stopped after Azure advised us they had made internal changes.

## Remediation

We believe there were two red herrings which turned out not to be contributing factors:

- The 404 errors from the WAF
- The CNAME flattening on the apex domain.
  In practice, removing the proxy and switching to AFD’s WAF are irrelevant to this issue - they are included here because they created a red herring when attempting to determine the causes.

Similarly, using CNAME flattening on the apex domain was muddying the actual causes. CNAME flattening is where the DNS provider creates a series of A Records instead of a single CNAME record to allow for dynamic resolution of the underlying IP address. This [explainer by Cloudflare](https://blog.cloudflare.com/introducing-cname-flattening-rfc-compliant-cnames-at-a-domains-root/) is a helpful resource for understanding.

Based on our best understanding of the problem, we created a www subdomain on the affected domains and created a CNAME record on this subdomain to give AFD authoritative control over the resolution of all requests to that domain. We then redirected traffic from the affected domains to the appropriate www subdomains.

This mitigation helped initially but the incorrect IP address resolution within the AFD endpoint reappeared. It was resolved after Azure advised that they had made changes to our AFD profile. We understand that Azure moved the affected AFD profile “from a legacy platform to the most updated platform”, but have not had official confirmation.

We considered rolling back the AFD profile changes once we understood the problem. We chose not to take this path as we understood that only a subset of the AFD nodes would update, leaving some regions in a broken state.

## Next steps

To help us understand in more depth, we have engaged with our Azure account manager on this issue.

Immediately following resolution, we addressed the impact of the www subdomain mitigation, particularly around CORS, static assets and SEO.

Once we were back to a stable state, we continued with the migration project as we still had records to migrate. We adapted the project to work closely with Azure to ensure that our changes work as expected. We continue to have scheduled downtime as we progress our migration. With the understanding that we now have, we can design rollback safe migration steps. This allows us to minimise the impact of scheduled downtime by rolling back immediately on unexpected behaviour.

We have run an incident review and have identified areas of improvement:
Development process: We are reviewing the quality controls in our development process to identify where we could have detected the problem before applying the changes to our production environment.

Incident response process for long-running incidents: We have not previously had an incident that ran for this long. We started to run out of rested on-call engineers with appropriate context to manage the incident. We are investigating how other internal and external groups manage these situations and are updating our on-call rosters and guidance accordingly.

## Conclusion

Octopus Deploy take service availability seriously. The time to resolution on this incident was longer than we aim for. We apologise for the disruption to our customers due to this outage.

We are continuing to investigate the complexities of this incident to ensure that our remediations will be effective, so our customers can focus on happy deployments.
