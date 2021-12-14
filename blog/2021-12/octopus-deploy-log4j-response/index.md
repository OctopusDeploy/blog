---
title: Octopus Deploy's Log4J Response
description: Octopus Deploy's response to the Log4J vulnerability (CVE-2021-44228)
author: kyle.jackson@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: blogimage-security.png
bannerImage: blogimage-security.png
bannerImageAlt: Octopus on shield to represent security
isFeatured: false
tags: 
  - Product
  - Trust and Security
---

![Octopus on shield to represent security](blogimage-security.png)

On Thursday, December 9, 2021, Octopus Deploy was made aware of a vulnerability in the Log4J logging utility ([CVE-2021-44228](https://cve.mitre.org/cgi-bin/cvename.cgi?name=2021-44228)). We immediately enacted our Security Incident Response Plan to investigate our usage of this utility and its impact across our products and infrastructure.

This post summarizes the results of our investigation to date and our recommended next steps for customers.

## JetBrains TeamCity plugin

Version 6.1.7 of the JetBrains TeamCity plugin has been updated to use version 2.15.0 of Log4J. However, all versions of the plugin before 6.1.7 are using version 2.14.1 of Log4J and could be vulnerable to remote code execution.


Version 6.1.7 of the Octopus Deploy plugin for TeamCity has been published to the marketplace. TeamCity users should be notified of this update through the Web UI and are strongly encouraged to update.

Follow our [security advisory](https://advisories.octopus.com/adv/2021-12---Octopus-Deploy-TeamCity-Plugin-log4j2-dependency.2306410241.html) for updates.

## Octopus Java SDK

Version 0.0.3 of the Java SDK has been updated to use version 2.15.0 of Log4J. However, all versions of the Java SDK before 0.0.3 are using version 2.14.1 of Log4J and could be vulnerable to remote code execution.

Customers using the Octopus Java SDK should update their SDK version to 0.0.3.

Follow our [security advisory](https://advisories.octopus.com/adv/2021-13---Octopus-Java-Client-SDK-log4j2-dependency.2306475696.html) for updates.

## Octopus Cloud, Octopus Server and Octopus Tentacle

Octopus Cloud, Octopus Server, and Octopus Tentacle are not affected by this vulnerability. They’re built with the .NET framework and don’t use any ported versions of Log4J.

## Breakdown of all products

| Product | Vulnerable Log4J Version Used | Vulnerable Product Versions | Patched Product Versions |
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

## Conclusion

We are continuing to track new updates relating to this vulnerability and will provide further updates if there is any new risk to our customers or products. 

If you have any questions please reach out via support@octopus.com.