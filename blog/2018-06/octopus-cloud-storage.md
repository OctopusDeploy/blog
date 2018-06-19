---
title: Octopus Cloud - Storage
description: Octopus Cloud's storage backend leverages SaaS solutions to offer optimum resiliency and availability
author: jason.brown@octopus.com
visibility: private
metaImage: metaimage-shipping-2018-6.png
bannerImage: blogimage-shipping-2018-6.png
published: 2018-06-20
tags:
 - OctopusCloud
---

Octopus Deploy has a couple of storage demands that we needed to solve for the release of Octopus Cloud. The first that will spring to mind for most existing Octopus Customers is the Database backend

Octopus Deploy moved to a SQL Server backend in version 3.0, switching from RavenDB to Microsoft's flagship database server.  

It allows us to easily consolidate multiple Octopus front-end instances onto shared SQL Server backends. We run SQL Server on Amazon's RDS platform, allowing us to consolidate multiple Octopus databases onto a single backend instance, and have our backups and transaction logs managed.

We get point-in time restore blah blah blah



File System storage was the other arm. This is not something most customers worry about about in detail, but for us, it was important that we found a reliable, resilient storage backend. Early iterations ran with Amazon EBS volumes, which in the case of a failed EC2 instance, could be re-attached to a new instance after failure. This was insufficient for several reasons. Firstly, because an EBS volume can only be attached to one instance, it limited our ability to provide HA and Load Balanced options within Octopus Cloud. It also means that even if we run a single instance or container per customer, we're limited in our ability to do a rolling replacement.

So, after much casting around for solutions, we finally hit upon AWS Storage Gateway.

Storage Gateway is designed as a flexible solution for storage in the cloud or a data center, as a bridge between cloud and on-premise. But in our case, Storage Gateway allows us to run Network file storage via NFS or iSCSI on EC2 instances and have that buffered and cached to S3 for resiliency. We can then mount those to our Octopus instances and hey presto, resilient, low cost, high capacity file storage.

The automation of Storage Gateway, however, isn't the simplest process in AWS, so we built a custom provisioning module that will:

- Using Cloudformation
- - Spin up a Storage Gateway instance 
- - Spin up ten accompanying S3 buckets
- - Spin up the accompanying IAM security policies
- - Spin up Security Groups and DNS records for each gateway
- Then Using PowerShell
- - Wait for the Cloudformation provisioning to complete
- - Register the Storage Gateway
- - Create ten share endpoints on each gateway and assign them correct firewall values
- - Bring the gateway into service on the Octopus Cloud Portal