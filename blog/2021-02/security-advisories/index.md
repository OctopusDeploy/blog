---
title: Introducing Octopus security advisories
description: Updating the way we keep our customers informed about product security
author: kyle.jackson@octopus.com
published: 2021-02-10
visibility: public
metaImage: blogimage-security.png
bannerImage: blogimage-security.png
bannerImageAlt: A stylized Octopus & Shield icon
tags:
  - Trust and Security
  - Product
  - Advisories
---

![A stylized Octopus & Shield icon](blogimage-security.png)

It's no surprise that software products have security vulnerabilities, or that when they do occur, it's important for vendors to have effective communication strategies to ensure systems are secure and users aren't left vulnerable.

Currently Octopus product security vulnerabilities are disclosed in CVE databases and in our product release notes, however, we believe we can enhance this communication to better protect our customers.

## How we currently notify customers about product security vulnerabilities

When we discover or are notified of a security vulnerability in our products we patch the vulnerability, create a CVE, create a [GitHub issue](https://github.com/OctopusDeploy/Issues), and update our release notes with the fix and the CVE number.

In terms of notifying our customers of security vulnerabilities we have always assumed our customers are either reading the release notes or are periodically checking CVE databases for any security vulnerabilities. However, we believe we can do more to help our customers have the information they need to ensure their environments remain secure.

To address this we're introducing Octopus security advisories.

## Octopus security advisories

Octopus security advisories are essentially the aggregation of the release notes, CVE details, and extra information we want to provide to our customers in one place so customers don't have to go hunting for information.

Some of the information provided in the security advisories will be:
- Vulnerability severity based on an Octopus Deploy assessment.
- Details about the vulnerability, including affected versions, versions fixed, and other relevant information.
- Whether there are any known exploits for the vulnerability that we are aware of.

## When will Octopus post a security advisory?

The Octopus Security Team will post a security advisory under the following circumstances:
- Preliminary advisories will be posted in circumstances where there is a critical or high risk vulnerability that impacts multiple Octopus products.
- With the release of a CVE and patch to fix a specific vulnerability in an Octopus product.
- If we believe our customers could be targeted by bad actors, for example providing guidance on identifying fake Octopus Deploy emails.

## Where will Octopus security advisories be posted?

Security advisories will be posted to [advisories.octopus.com](https://advisories.octopus.com), and a link to the security advisory will be posted to the [Octopus Deploy Twitter](https://twitter.com/OctopusDeploy) feed.

## More information

For more information about Octopus Deploy's security journey through 2021 you can see our [Trust & Security Roadmap](https://github.com/OctopusDeploy/Issues/issues/6523).

If you need to report a vulnerability please contact us at [security@octopus.com](mailto:security@octopus.com).
