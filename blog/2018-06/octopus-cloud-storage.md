---
title: Octopus Cloud - Storage
description: Octopus Cloud's storage backend leverages SaaS solutions to offer optimum resiliency and availability
author: jason.brown@octopus.com
visibility: private
metaImage: metaimage-shipping-2018-6.png
bannerImage: blogimage-shipping-2018-6.png
published: 2018-06-20
tags:
 - Engineering
---

Octopus Deploy has a couple of major storage demands that we needed to solve for the release of Octopus Cloud. The first that will spring to mind for most existing Octopus Customers is the Database backend, but of equal importance is the disk, where your config files, logs, artifacts and packages live.

## Database

The database backend is a foundational part of the Cloud project, so it was given a lot of attention early on. Initial prototypes were run against shared Amazon RDS instances, but we also explored the possibility of running on managed offerings from other cloud providers.

We also toyed with running our own database backends, at various scales from a single instance per-customer to large consolidated instances serving many customers at once. 

It was decided eventually that sheer management overhead, along with the initial investment in automation, made running our own custom SQL Backend a poor choice. Amazon RDS's automatic backups, managed resiliency and point-in-time restore capabilities really made up for the slight premium we'd be paying in pure dollar value.

Indeed the point-in-time backup capability showed its value during the Alpha Phase, when a Cloud Architect Who Shall Remain Nameless* ran an unsanitised query against our Octopus Hub instance and hosed the entire deployment history. Point-in-time restore meant we were back to full operation after only a brief outage, with a mere four lost rows needing to be reconciled manually.

We can still consolidate multiple databases onto a single instance, offsetting costs without significant performance impacts, and our testing so far has been extremely positive with regard to DB performance.

This kind of assurance is of enormous value to us, and by extension to our customers.

As well as making sure your instances are up and stay up, we want to make sure your data is protected from prying eyes as much as possible. Every instance connects to its database across private, secured subnets using a unique account tied solely to that customer, and only allowed access to the specific database. We also encrypt our backups at rest using S3's KMS encryption support, and we limit access to those backups.

Lastly, RDS is extremely monitorable, with metrics piped into CLoudwatch and from there to our dashboards and alerting. An overworked Database Server will trigger a cascade of information into our support channels, and we can take appropriate action.

## File System

File System storage was the other part of the storage picture. We need to safely store things like Artifacts, Packages and TaskLogs, in a way that is both resilient to failure (cloud instances are far from infallable) and responsive.

Early sprototypes ran with simple Amazon EBS volumes, which in the case of a failing EC2 instance, could be re-attached to a new instance after failure. This was good, but insufficient for several reasons. Firstly, because an EBS volume can only be attached to one instance at a time, it limited our ability to provide HA and Load Balanced options within Octopus Cloud. It also meant that even if we run a single instance or container per customer, we were limited in our ability to do a rolling replacement of a failing instance. No rolling replacement means an outage to replace an impaired instance, and that's not ideal.

So, after much casting around for solutions, we finally hit upon AWS Storage Gateway.

Storage Gateway is designed as a flexible solution for storage in the cloud or a data center, and is often used as a bridge between cloud and on-premise data centers.

But in our case, Storage Gateway allows us to run Network file storage - via NFS or iSCSI attachment - on EC2 instances, and have that buffered and cached to S3 for resiliency. We just mount those to our Octopus instances and hey presto, resilient, low cost, high capacity file storage isolated from failure of the instance itself.

This means we can treat the instances themselves as somewhat disposable, with rolling replacements a genuine option, and the ability to do quick, reliable automatic recovery just by connecting to a network location.

The automation of Storage Gateway, however, isn't the simplest process in AWS, so we built a custom provisioning module that runs in our Octopus Hub. The process looks a little like this:

- Using Cloudformation
- - Spin up a Storage Gateway instance 
- - Spin up ten accompanying S3 buckets
- - Spin up the accompanying IAM security policies
- - Spin up Security Groups and DNS records for each gateway
- Then Using PowerShell
- - Wait for the Cloudformation provisioning to complete
- - Register the Storage Gateway with AWS
- - Create ten share endpoints on each gateway and assign them correct firewall values
- - Tag each new endpoint as "Available"
- - Bring the gateway into service on the Octopus Cloud Portal

Each Storage Gateway instance supports ten customer endpoints, each backed by a single S3 bucket, with its own policy restricting access. The endpoint itself is firewalled using security group rules to the IP address of the customer instance, to prevent leaks and cross-contamination. Should an instance be decommissioned, the data is zipped up and sent off to deep storage, then the share is scrubbed clean and returned to the pool.

In the case of a gateway failure, we can quickly replace the EC2 instance, and reassociate the S3 buckets with their gateway, with no loss of data.

## Pooling

Because both these solutions support multiple customers per-instance, we need to maintain a pool of available capacity. This is monitored and managed via our Hub Octopus instance, which runs a check for available capacity on every instance spin-up, and provisions fresh capacity when we're running low. We want to do this well in advance of actually running out, as the lag time for a new RDS instance can be non-trivial. One of our aims is for signup to be as smooth as possible, and we'll be tweaking our thresholds throughout the launch phase to keep wait times low.

## In summary

Storage can look like a relatively simple piece fo the puzzle, but at scale it becomes a much more involved problem. We've put in a lot of work to ensure your data is safe, secure, and available when you need it.


Happy (Octopus Cloud) deployments!

* it was me