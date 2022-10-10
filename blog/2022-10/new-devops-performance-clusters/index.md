---
title: The new DevOps performance clusters
description:
author: steve.fenton@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - DevOps
---

The Accelerate State of DevOps Report contained a surprising change to the traditional performance clusters. The report also introduced a new way to group organizations using a different approach to clustering.

This post introduces you to the traditional performance groups, explains this year's changes, and describes the new approach to clustering.

## Previous changes

Some previous adjustments to the report have a bearing on the new changes that emerged this year. In past reports, 4 key metrics were used to predict the performance of an organization:

 - Throughput

 1. Deployment frequency (DF)
 1. Lead time for changes (LT)

 - Stability

 3. Change failure rate (CFR)
 3. Mean time to recovery (MTTR)

In 2018, a fifth metric was added for *availability*. The researchers later changed this to *reliability*. The original 4 metrics represented software delivery performance, but with the added reliability metric, the complete set measures software delivery and operational performance (often shortened to "SDO performance").

You'll often find statements that mention the relationship between the SDO performance metrics and broader organizational performance.

Teams who excel in all five measures exhibit exceptional organizational performance.

## Software delivery performance clusters

If you've followed the report for a while, you'll be familiar with the software delivery performance clusters. Each group represents a different level of performance against throughput and stability metrics. This usually results in four clusters:

| Performance level | Lead time      | Deployment frequency           | Change failure rate | Mean time to resolve |
|-------------------|----------------|--------------------------------|---------------------|----------------------|
| Elite             | < 1 hour       | Multiple times per day         | 0-15%               | < 1 hour             |
| High              | 1 day - 1 week | Weekly to monthly              | 16-30%              | < 1 day              |
| Medium            | 1-6 months     | Monthly to biannually          | 16-30%              | 1 day - 1 week       |
| Low               | > 6 months     | Fewer than once every 6 months | 16-30%              | > 6 months           |

Organizations can assess their performance based on the same measures and identify potential improvement areas.

## What changed in 2022?

- The elite cluster has gone
- The high-performance group shrank
- The medium group is performing better than before
- The low cluster is performing better than before but has grown


Different people respond. This usually doesn't change the insights if the groups of respondents each year reflects the target population. However, there was a big demographic change this year with fewer experienced respondents.

![Respondents in 2022 had less experience than last year](years-of-experience.png)

This could mean either:

- Past surveys have been skewed towards higher levels of experience because those were the organizations interested in DevOps
- The 2022 survey may be skewed towards less experience as the researchers wanted to get more responses from individual contributors
- A little of both




The latest [Accelerate State of DevOps Report] is missing the *elite* cluster. The data collected in 2022 didn't have a clear gap between hight and elite performers, so they had to be considered a single group.

The performance of the high performance group increased, but 



You might be surprised by this change, so it worth looking at the re




  - In 2022, just low, medium, and high (elite and high rolled into one)
    - The high cluster (11%) performs between previous high and elite levels (as expected) but has got substantially smaller
    - The medium cluster is performing better than previously, and is the largest cluster (69%)
    - The low cluster is performing a little better than before, but has also grown (19%)
    - Yes, it adds up to 99% and also... it kinda depends on who writes back each year... so make sure you respond next year to give us all the most complete picture
    - Key fact... high performers deploy 417x more often than low performers!
  - Cluster table based on original 4 DORA metrics



- What is a cluster
  - You throw down all the different organizations onto a chart and the clusters appear
  - Organizations in a cluster are all similar to each other
  - Organizations in one cluster are different to organizations in other clusters
  - You create the clusters *after* the data, so they can change... they appear from the data



- The new clusters are based on all 5 SDO metrics
  - Created because operations capability is critical for software delivery performance - organizations with great software delivery potential don't get results if the operations capability isn't there
  - Less judgmental, more descriptive
  - Starting, flowing (17%), slowing (>33%), retiring
  - You should take action if your plan doesn't align to the cluster description, if you find yourself in the Slowing or Retiring cluster for a core software product, you need to take action
  - If you are retiring software and find it ranks as retiring in the model, that's to be expected
  - Bear in mind how Google operates, they have lots of software products in various stages of their lifecycle, from established search and advertising products, to flagship cloud products, to long-tooth products that are destined to follow Google Reader and Google+. If you are working at a product company, you might have one really critical product and you really need to be in the flowing state. For line of business apps you might happy to cruise in slowing mode if the apps aren't creating a core differentiation from competitors, but otherwise you'll want to aim for flowing too.
  - *If your cluster doesn't match your product ambition, you need to pay attention to that and change*

- The flowing state
  - The following are more common in this cluster
    - Loosely coupled architectures
    - Flexible work arrangements
    - Version control and change management (code and configuration)
    - Continuous integration - more frequent integrations to the main branch
    - Continuous Delivery - they get changes into production safely, sustainably, and efficiently
    - Unlike other clusters, they manage high SDO performance without documentation, other clusters perform less well unless they have more documentation
    - The flowing cluster most needs generative culture and organizational performance suffers without it

    - Other clusters may achieve an adequate performance level, but at a far greater cost and with burnout, unplanned work, and low rates of software change... the retiring cluster is where large organizations with an ops team that reign supreme exist... no changes happen without their approval, so things are reliable, but slow and people will burn out (DENPLAN :)) A large organization is likely to be successful for reasons other than technology
    - It is likely more research is needed to further understand the clusters

- The real measures are organization wide, real-world performance against your commercial and non-commercial goals

Happy deployments!