---
title: "Removing support for SHA-1 certificates in Octopus Tentacles"
description: Learn about upcoming changes to deprecate and remove support for SHA-1 certificate in Tenacles, improving security and enhancing support for modern operating systems and runtime environments.
author: simon.canning@octopus.com
visibility: public
published: 2025-03-26-1400
metaImage: security.png
bannerImage: security.png
bannerImageAlt: Blue DevOps infinity diagram outlined in green with a green security shield over the right top-hand corner.
isFeatured: false
tags: 
  - Product
  - Trust and Security
  - Octopus Cloud
---

In 2017, we [discussed](https://octopus.com/blog/shattered) the implications of the [SHA-1 Shattered Collision](https://shattered.io/) and what that meant for the Octopus Server to Tentacle communication channel. At the same time, we made [product updates](https://octopus.com/docs/security/cve/shattered-and-octopus-deploy) to ensure that SHA-256 was used when new self-signed certificates were generated for versions of [Octopus 3.14](https://octopus.com/blog/octopus-release-3-14). 

We've now decided to remove SHA-1 support. Fortunately, most customers use SHA-256 certificates, but some still use SHA-1 certificates. 

In this post, I outline how this deprecation affects different customers. I also provide a timeline of what to expect and how to regenerate new SHA-256 certificates.

## Understanding communication between Octopus Server and Tentacles

Security is paramount in modern software deployment environments. Octopus implements strong security measures for communication between the Octopus Server and its Tentacles. 

Here’s a deep dive into how this communication is established, the role of TLS, and the rationale behind using self-signed certificates out of the box.

### Secure communication via TLS

When Octopus Server communicates with its Tentacles—the agents that handle deployment tasks—this interaction is always conducted over a secure Transport Layer Security (TLS) connection. TLS ensures the data exchanged between the 2 entities remains confidential and tamper-proof. Each server and Tentacle possess a unique public/private key pair crucial to establishing a secure and trusted link.

The Octopus Server’s thumbprint gets sent to the Tentacle during the configuration process. This thumbprint is a unique identifier for the Server's certificate, establishing a trusting relationship between the Server and the Tentacle. Conversely, the Tentacle informs the Octopus Server of its thumbprint. As a result, commands issued by the server get accepted exclusively by those Tentacles it recognizes and trusts, and vice versa.

This setup ensures that only legitimate entities can issue or accept commands, significantly enhancing the security of your deployment processes.

### The use of self-signed certificates

A common question about Octopus Deploy’s security model is why it defaults to using self-signed certificates instead of certificates issued by a Certificate Authority (CA). At first glance, using self-signed certificates may seem less secure, but there are several compelling reasons for this choice:

1. *Customized trust relationships*: The self-signed approach allows for precise trust management. When you set up a Tentacle, you manually input the Octopus Server's thumbprint. This process means that every connection is verified individually, ensuring authenticity. The connection is aborted if the server's identity cannot be confirmed through its thumbprint.
2. *Operational efficiency*: Using self-signed certificates simplifies the installation and setup, particularly in automated environments. Instead of relying on a CA to generate certificates, which can introduce delays and complexities, Tentacles can quickly generate their certificates upon installation. Additionally, importing existing certificates can be streamlined, facilitating the automation of Tentacle installation.
   
:::hint
We're open to feedback about how this decision affects your ability to manage your Octopus Server and Tentacles at scale. We're particularly interested in hearing from customers who'd like to discuss their current credential management and rotation practices.
:::

### How it all comes together

When a Tentacle is registered with the Octopus Server, the 2 entities exchange their thumbprints, identifying their certificates. This process lays the foundation for a secure and trusted relationship. When the Octopus Server connects to a Tentacle, it checks that the Tentacle presents a valid certificate matching the expected thumbprint. This verification process is reciprocal; the Tentacle similarly checks that the connection originates from a trusted Octopus Server. If any discrepancies arise, the connection is immediately rejected. 

This model mirrors principles in other secure communication protocols, like SSH, used in UNIX systems, providing a robust and familiar security framework.

## How can I determine if I’m affected?

Below is an example command that will print out the machines (deployment target Tentacles) with sha1RSA set as their certificate signature algorithm. You'll need to replace the `instance-name` with your instance name or the full URL if you’re a self-host customer, `space-id` with a space ID (e.g., `Spaces-1`), and replace the API key with a key that has at least the MachineView permission.

```
curl -H "X-Octopus-Apikey: API-XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX" -H "Accept: application/json" "https://<instance-name>.octopus.app/api/<space-id>/machines" |  jq '.Items .[] | select(.Endpoint.CertificateSignatureAlgorithm=="sha1RSA")'
```

For more detail, read about [using the Octopus API with Bash and jq](https://octopus.com/blog/api-bash-jq#using-the-octopus-api-with-bash-and-jq). Alternatively, you can [interact with the Octopus Deploy API using PowerShell](https://octopus.com/blog/interacting-with-the-octopus-deploy-api-using-powershell).

## Do I have to wait for a new Octopus Server or Tentacle version?

No, you do not. You can take action now. Octopus has supported SHA-256 certificates for Server to Tentacle communications since the [3.14 release in June 2017](https://octopus.com/blog/octopus-release-3-14).

## What do I need to do?

To configure a Tentacle to use a new certificate, ensure your Tentacle version is on the latest version and [follow the instructions in our docs](https://octopus.com/docs/security/octopus-tentacle-communication/regenerate-certificates-with-octopus-server-and-tentacle#ConfiguringATentacleToUseANewCertificate).

## What is the deprecation schedule?

Below are details of the deprecation schedule, depending on the Octopus product and operating system your server runs on.

### Octopus Cloud customers

We plan to start emailing affected customers about the deprecation on March 31, 2025, and give at least 60 days' notice before we switch off support for SHA-1 on June 1, 2025. After that point, any deployment target or worker still using SHA-1 Tentacle will no longer be able to connect to Octopus Cloud. We plan to give reminders regularly, including 30 days, 7 days, and a final warning. You can take action now.

### Octopus Server Linux container customers

We plan to release an in-product configuration option to turn on/off the SHA-1 Tentacle support in Octopus Server 2025.2 and remove support for SHA-1 Tentacles in the subsequent Octopus Server 2025.3 release. These changes will impact Octopus Server Linux container customers only. 

As a self-hosted customer, you control the update cycle of your Octopus Server and, therefore, have some flexibility around how and when to perform the appropriate updates. We recommend you take action as soon as possible.

### Octopus Server Windows customers

There are no immediate required actions for Octopus Server Windows self-hosted customers. This is because we can continue to support SHA-1 certificates using supported operating systems and runtime environments for the time being. 

As a self-hosted customer, you control the update cycle of your Octopus Server and, therefore, have some flexibility around how and when to perform updates. We still recommend you take action and move off SHA-1 certificates as soon as possible.

### Future work

In the future, we expect to add features in Octopus Server to support automatically regenerating certificates on the Octopus Server and all Tentacles. We're currently in the discovery phase of this work. We're particularly interested in hearing from customers who would like to discuss their current credential management and rotation practices.

## Conclusion

Removing support for SHA-1 certificates in Octopus Tentacles is part of our commitment to deliver secure software deployment environments. It enables us to continue to support modern operating systems and runtime environments. 

While most customers have moved to the more secure SHA-256 certificates, it's crucial for those still using SHA-1 to understand the implications of this deprecation. The communication between Octopus Server and Tentacles relies on secure TLS connections and self-signed certificates, promoting customized trust relationships and operational efficiency. As the move towards stronger cryptographic algorithms continues, we encourage customers to regenerate their certificates to align with best practices. 

We welcome feedback about credential management and practices to ensure the Octopus platform meets your needs effectively. This transition bolsters security measures, further safeguarding the deployment processes critical to businesses.

Happy deployments!