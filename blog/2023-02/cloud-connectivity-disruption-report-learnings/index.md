---
title: "Octopus Cloud connectivity disruption - report and learnings"
description: Public incident report and our learnings about the Octopus Cloud network connectivity disruption from January 25, 2023.
author: alix.klingenberg@octopus.com
visibility: public
published: 2023-02-01
metaImage: blogimage-incidentazurenetworkoutage-2023x2.png
bannerImage: blogimage-incidentazurenetworkoutage-2023x2.png
bannerImageAlt: A stylized Octopus and shield icon
isFeatured: false
tags:
  - Engineering
  - Incident Reports
---

Between 07:05am UTC and 12:43am UTC on Wednesday, January 25, 2023, Octopus Cloud customers in some regions experienced network connectivity issues. This caused long network latency and/or timeouts when they tried to connect to their Octopus Cloud instance, and impacted some functions of Octopus Deploy. In particular, some Octopus Deploy operations would fail if that operation involved a network call over the impacted network. This outage impacted Octopus Cloud, Octopus.com/OctopusID, and the octopus.com Control Center.

A network outage in our upstream cloud provider, Azure, caused the disruption.

In this post, we detail what happened and our learnings.

As a result of this incident, we're strengthening our relationship with Azure and improving our monitoring of their availability.

## Key timings

| Event                        | Time period |
| ---------------------------- | ----------- |
| Time to detection            | 40 mins     |
| Time to incident declaration | 69 mins     |
| Time to resolution           | 4hrs 32mins |

## Incident timeline

Wednesday, January 25, 2023 _(all dates and times below are shown in UTC)_

**_7:45:_** An Octopus engineer noted access to Azure’s portal was down due to request timeouts. Azure’s status page was still green (working) for all services. The engineer identified it was likely the connectivity issues were in Azure’s internal networks regardless of the status page, as it doesn't always accurately reflect the current status. They also noted it was possible the network issue was preventing Azure from updating their status page. The engineer noted an increased load on the database used by octopus.com/OctopusID, and there was a risk of customer-facing login issues if it continued.

**_7:51:_** A support engineer noted that a customer reported being unable to log in using OctopusID approximately an hour prior.

**_7:51 - 8:14:_** Octopus engineers continued to monitor internal services and noted similar issues with external services. All observed issues were consistent with a cloud provider network outage.

**_8:14:_** An Octopus engineer declared an incident and started the incident response process.

**_8:14 - 8:19:_** On-call engineers investigated the customer impact and updated [our status page](https://status.octopus.com/) to reflect the partial outage for octopus.com/OctopusID, Octopus Cloud and the Cloud Control Center. They also paused any internal processes (for example, deployments of Octopus Server to Octopus Cloud) that could potentially exacerbate the incident.

**_8:33:_** An on-call engineer noted that Azure’s status page now showed Azure investigating significant network issues across all regions.

**_8:33 - 8:42:_** On-call engineers continued to monitor the upstream network outage and guided support teams to point impacted customers to our status page.

**_8:42:_** Based on the continued outage and significant customer impact, the outage was upgraded to major on Octopus’s status page.

**_9:33_** - 9:59: On-call engineers noted status updates from Azure indicating the cause of the outage was identified and resolution of the incident was in progress.

**_10:57:_** On-call engineers confirmed with support engineers that customers were starting to see their connectivity restored to the impacted Octopus services. Engineers confirmed that internal services also affected by the upstream outage were starting to recover.

**_11:37:_** The incident was declared resolved and the status page was updated accordingly.

## Technical details

Upstream cloud provider outages are notoriously difficult to resolve for their customers, because the issue is usually out of the control of the customer.

Azure’s internal network carries the traffic from various Octopus services that run in the cloud. In this case, the affected services were Octopus Cloud, octopus.com/OctopusID and the Cloud Control Center. 

The impact was primarily to external and cross-region traffic. We're reliant on that network to carry that traffic and don't plan to build fallback options for this particular cloud provider service.

### Azure’s technical details

> We determined that a change made to the Microsoft Wide Area Network (WAN) impacted connectivity between clients on the internet to Azure, connectivity across regions, as well as cross-premises connectivity via ExpressRoute. As part of a planned change to update the IP address on a WAN router, a command given to the router caused it to send messages to all other routers in the WAN, which resulted in all of them recomputing their adjacency and forwarding tables. During this re-computation process, the routers were unable to correctly forward packets traversing them. The command that caused the issue has different behaviors on different network devices, and the command had not been vetted using our full qualification process on the router on which it was executed.”

Source: https://status.azure.com/en-us/status/history/, retrieved on Tuesday, January 31, 2023.

## Remediation

Octopus takes service availability seriously. Despite the difficulty with upstream cloud provider outages, we fully review and remediate any outages that occur. We do this so that we're continuously improving and maintaining the best possible service we can.

### Cloud provider relationship

We engaged with Azure to review our escalation process options when raising an issue with them. We intend to follow this up by aligning our internal escalation processes with their recommendations, so we streamline all communication with Azure about outages.

### Internal monitoring

Our availability monitoring caught this issue, but because the network issues were sporadic, the failures didn't meet our internal thresholds for alerting. We're reviewing our incident response guidance to more closely monitor our cloud provider’s availability dashboards.

## Conclusion

We apologize to our customers for any disruption and inconvenience. We've taken steps to refine our issue escalation process with our upstream cloud provider. We're also reviewing our instrumentation and alerting around network outages from Azure so we can communicate impact to our customers early and effectively, so they can focus on happy deployments.
