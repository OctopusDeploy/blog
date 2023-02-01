---
title: Network Connectivity Disruption on January 25, 2023
description: Public incident report on network connectivity disruption from January 25, 2023
author: alix.klingenberg@octopus.com
visibility: public
published: 2023-02-01-1800
metaImage: blogimage-devopsandplatformengineering-2022x1.png
bannerImage: blogimage-devopsandplatformengineering-2022x1.png
bannerImageAlt: Stylized image of DevOps infinity symbol with a car driving on it and increasing speed over golden arrows.
isFeatured: false
tags:
  - Engineering
  - Incident Reports
---

## Summary

Between 07:05am UTC and 12:43am UTC on Wednesday, 25 January, 2023, Octopus Cloud customers in some regions experienced network connectivity issues, manifesting as long network latency and/or timeouts when trying to connect to their Octopus Cloud instance, and impacting some functions of Octopus Deploy. In particular, some Octopus Deploy operations would fail if that operation involved a network call over the impacted network. This outage impacted Octopus Cloud, Octopus.com/OctopusID and the Octopus.com Control Center.

This disruption was caused by a network outage in our upstream cloud provider, Azure.
As a result of this incident, we are strengthening our relationship with Azure and improving our monitoring of their availability.

### Key Timings

| Event                        | Time period |
| ---------------------------- | ----------- |
| Time to detection            | 40 mins     |
| Time to incident declaration | 69 mins     |
| Time to resolution           | 4hrs 32mins |

## Incident Timeline

Thursday, 25 January, 2023 _(all dates and times below are shown in UTC)_

_7:45:_ An Octopus engineer notes that access to Azure’s portal is down due to request timeouts. Azure’s status page is still green (working) for all services. The engineer identifies that it is likely that the connectivity issues are within Azure’s internal networks regardless of the status page, as it does not always accurately reflect the current status. They also note that it is possible that the network issue is preventing Azure from updating their status page. The engineer notes that we are starting to see increased load on the database used by Octopus.com/OctopusID, and that there is a risk of customer-facing login issues if it continues.

_7:51:_ A support engineer notes that a customer reported being unable to log in using OctopusID approximately an hour ago.

_7:51 - 8:14:_ Octopus engineers continue to monitor internal services and note similar issues with external services. All observed issues are consistent with a cloud provider network outage.

_8:14:_ An Octopus engineer declares an incident and starts the incident response process.

_8:14 - 8:19:_ On-call engineers investigate the customer impact and update [our status page](https://status.octopus.com/) to reflect the partial outage for Octopus.com/OctopusID, Octopus Cloud and the Cloud Control Center. They also pause any internal processes (e.g. deployments of Octopus Server to Octopus Cloud) that could potentially exacerbate the incident.

_8:33:_ An on-call engineer notes that Azure’s status page now shows that they are investigating significant network issues across all regions.

_8:33 - 8:42:_ On-call engineers continue to monitor the upstream network outage and provide guidance to support teams to point impacted customers to our status page.

_8:42:_ Based on the continued outage and significant customer impact, the outage is upgraded to major on Octopus’s status page.

_9:33_ - 9:59: On-call engineers note that there are status updates from Azure indicating that the cause of the outage has been identified and that resolution of the incident is in progress.

_10:57:_ On-call engineers confirm with support engineers that customers are starting to see their connectivity restored to the impacted Octopus services. Engineers confirm that internal services also affected by the upstream outage are starting to recover.

_11:37_ The incident is declared resolved and the status page is updated accordingly.

## Technical Details

Upstream cloud provider outages are notoriously difficult to resolve for their customers as the issue is usually out of the control of the customer.

Azure’s internal network carries the traffic from various Octopus services that run in the cloud. In this case, the specific affected services were Octopus Cloud, Octopus.com/OctopusID and the Cloud Control Center. The impact was primarily to external and cross-region traffic. We are reliant on that network to carry that traffic and we do not plan to build fallback options for this particular cloud provider service.

### Azure’s Technical Details

> “We determined that a change made to the Microsoft Wide Area Network (WAN) impacted connectivity between clients on the internet to Azure, connectivity across regions, as well as cross-premises connectivity via ExpressRoute. As part of a planned change to update the IP address on a WAN router, a command given to the router caused it to send messages to all other routers in the WAN, which resulted in all of them recomputing their adjacency and forwarding tables. During this re-computation process, the routers were unable to correctly forward packets traversing them. The command that caused the issue has different behaviors on different network devices, and the command had not been vetted using our full qualification process on the router on which it was executed.”
> Source: https://status.azure.com/en-us/status/history/, retrieved on Tue 31 Jan 2023.

## Remediation

Octopus takes service availability seriously. Regardless of the disclaimer above regarding the difficulty of upstream cloud provider outages, we fully review and remediate any outages that occur. We do this in order to ensure that we are continuously improving and maintaining the best possible service we can.

### Cloud Provider Relationship

We have engaged with Azure to review our escalation process options when raising an issue with them. We intend to follow up this engagement with aligning our own internal escalation processes with their recommendations, to ensure that all communication with them regarding outages is streamlined.

### Internal Monitoring

Our availability monitoring caught this issue, but as it was sporadic and did not meet our internal thresholds for alerts, we did not get alerted. We are reviewing our incident response guidance with the intention to more closely monitor our cloud provider’s availability dashboards.

## Conclusion

We apologise to our customers for any disruption and inconvenience. We have taken steps to refine our issue escalation process with our upstream cloud provider. We have also undertaken to review our instrumentation and alerting around network outages from Azure with the intention of being able to communicate impact to our customers early and effectively so that they can focus on happy deployments.
