---
title: Octopus Deploy's response to the OpenSSL vulnerability
description: Important information regarding CVE-2022-3786 for Octopus Deploy.
author: colby.prior@octopus.com
visibility: public
published: 2022-11-03
metaImage: blogimage-devsecopsvulnerabilities-2022.png
bannerImage: blogimage-devsecopsvulnerabilities-2022.png
bannerImageAlt: Blue DevOps infinity diagram outlined in green with a green security shield over the right top-hand corner.
isFeatured: false
tags:
  - Product
  - Trust and Security
---

This week, there was a high severity vulnerability [announced by OpenSSL](https://www.openssl.org/blog/blog/2022/11/01/email-address-overflows/) that affects versions 3.x. 

We can confirm that Octopus Deploy depends on OpenSSL versions 1.x, which are not affected by this vulnerability. The OpenSSL vulnerability only affects servers configured to validate client certificates.

- Client certificate authentication is used by [Octopus Tentacle Communication](https://octopus.com/docs/security/octopus-tentacle-communication#Octopus-Tentaclecommunication-Octopus/Tentacletrustrelationship) to register with the Octopus Deploy Server. This is not affected because it currently depends on versions of OpenSSL 1.x.
- The OpenSSL version Octopus Tentacle is using is under support until 11th September 2023.

For more details on the OpenSSL vulnerability, check out this excellent write up [published on the Datadog blog](https://securitylabs.datadoghq.com/articles/openssl-november-1-vulnerabilities/).