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

Some previous adjustments to the report have a bearing on the new changes that emerged this year. In past reports, DORA (DevOps Research and Assessment) researchers used 4 key metrics to predict the performance of an organization:

 - Throughput

 1. Deployment frequency
 1. Lead time for changes

 - Stability

 3. Change failure rate
 3. Mean time to recovery

In 2018, DORA added a new metric for *availability*. They later changed this to *reliability*. You could measure software delivery performance with the original 4 metrics, but using all 5 lets you measure software delivery *and operational* performance, often shortened to *SDO performance*.

Performing well against all 5 measures drives organizational performance. The organization is more likely to achieve its goals where teams excel in these metrics.

## Clusters

A cluster is a group of data whose members are more similar to each other than they are to items in another group. This is useful in the State of DevOps report as it allows many different organizations to be arranged methodically for comparison.

The research team uses hierarchical clustering to discover the performance groups in the State of DevOps report. This technique doesn't define clusters in advance but lets them emerge from the data.

These clusters are used to test hypotheses, such as "Continuous Delivery drives software delivery performance". Each time the researchers repeat the analysis on a different sample, they can test the validity of the hypothesis.

## Software delivery performance clusters

If you've followed the State of DevOps Report for a while, you'll be familiar with the software delivery performance clusters. Each group represents a different level of performance against throughput and stability metrics. This usually results in four groups:

| Performance level | Lead time      | Deployment frequency           | Change failure rate | Mean time to resolve |
|-------------------|----------------|--------------------------------|---------------------|----------------------|
| Elite             | < 1 hour       | Multiple times per day         | 0-15%               | < 1 hour             |
| High              | 1 day - 1 week | Weekly to monthly              | 16-30%              | < 1 day              |
| Medium            | 1-6 months     | Monthly to biannually          | 16-30%              | 1 day - 1 week       |
| Low               | > 6 months     | Fewer than once every 6 months | 16-30%              | > 6 months           |

Although the same names are used for the clusters each year, the overall performance of a cluster changes. For example, if you take an organization from the center of the medium performance group, you'll find their performance differs each year. Organizations at the edge of a cluster might move up or down each year. The clusters emerge from the data, not the other way around.

Organizations can assess their performance based on the same measures and identify potential improvement areas.

### How Software delivery clusters changed in 2022

There are changes for all 4 clusters in the 2022 report. The most dramatic difference is that the elite cluster failed to emerge from the data, so the researchers had to remove it from this year's report. The other groups also changed:

- The high-performance group was smaller
- The medium group is performing better than before
- The low group is performing better than before but has grown

In the chart below, you'll find a comparison of 2021 and 2022. The performance axis shows how the clusters have moved over the past year. The medium and high clusters are both performing better. The circles indicate the number of organizations in each cluster.

![Comparison of performance](2021-2022-performance.png)

You can draw some conclusions from this chart, such as the high-performance group has fewer organizations but performs better. Although the medium cluster is much larger, it also performs better than last year.

Further research will be needed to understand the cause of the changes to clusters, but it is likely to relate to a demographic shift in respondents.

In previous years, many respondents had been in software delivery for a decade or more. In 2022 the proportion of respondents with more than 10 years of experience fell by half. The change seems less surprising as most respondents have less experience than those answering the surveys in prior years.

![Respondents in 2022 had less experience than last year](years-of-experience.png)

While you can see the clear change in demographics, what isn't known is whether this makes the sample more or less representative of the software development industry. However, as an individual looking at the insights, you can now think about how the level of experience in your team might affect your performance. You may be more likely to be an elite performer if your team has more members with a decade or more of software development experience.

Removing the elite performance category doesn't mean these organizations gave up software development. It tells us that this year's respondents had less experience overall and performed less well against the throughput and stability metrics.

Despite dramatic changes, one thing that remains the same is the gap between high and low performers. The report still shows that software delivery performance predicts organizational outcomes.

## The SDO performance clusters

The new clusters for SDO performance use all 5 metrics for throughput, stability, and operational performance:

 - Throughput

  1. Deployment frequency
  2. Lead time for changes

 - Stability

  3. Change failure rate
  3. Mean time to recovery

- Operational performance

  5. Reliability

The reliability metric captures how often the team achieves its reliability targets. Each group's label suggests a scenario where you might expect the performance level. For example, the "starting" cluster exhibits performance you'd expect from a team starting out with modest performance against each dimension. In the early stages of development, the team may be focused more on innovation than reliability.

Here are the SDO performance clusters:

| Cluster  | Lead time        | Deployment frequency       | Failure rate | MTTR           | Reliability   |
|----------|------------------|----------------------------|--------------|----------------|---------------|
| Starting | 1 week - 1 month | Weekly or monthly          | **31-45%**   | 1-7 days       | **Sometimes** |
| Flowing  | < 1 day          | On demand                  | 0-15%        | < 1 hour       | Usually       |
| Slowing  | 1 week - 1 month | Weekly or monthly          | 0-15%        | < 1 day        | Usually       |
| Retiring | **1-6 months**   | **Monthly or bi-annually** | **46-60%**   | **1-6 months** | Usually       |

These groups are more descriptive than the software delivery performance clusters. Rather than striving for high performance across all products and teams, an organization can take a more balanced approach. An organization can plan for different performance levels from teams working on their core product and others working on a new product.

This approach isn't without its hazards. Many teams remain in the starting group permanently, never resolving the reliability issues that would unlock their software delivery performance. Equally, Not all teams in the retiring cluster are in the process of shutting down a system. The same performance characteristics would be present in an organization where the operations team acts as gatekeepers, preventing changes from moving through the system.

You should view the cluster names as best-fit explanations for deliberate performance choices. You might choose to give up some reliability to encourage risk-taking and innovation on a new product concept. If you don't make intentional choices, you risk inventing a narrative to explain poor performance. Future research may test how accurate the cluster names are.

The most practical application of the new clusters is to use them to focus your efforts on your core software systems. The systems you sell and the line-of-business applications that give you a competitive advantage are the ones to bring into a flowing state. Never classify these systems as *slowing*, and only designate a system as *retiring* when you have an active plan for its withdrawal or replacement.

Teams in the flowing state more commonly use the capabilities of the DevOps structural equation model, such as:

    - Deployment automation
    - Loosely coupled architectures
    - [Continuous Delivery](https://octopus.com/devops/continuous-delivery/)
    - Version control
    - Flexible work arrangements

While some organizations achieve adequate performance without flowing software delivery and operations, they do so at greater cost with more burnout, unplanned work, and lower change rates.

## Conclusion

The disappearance of the elite performance cluster may be related to the change in demographics.

The new SDO performance clusters need further research and analysis but provide an interesting way to focus continuous improvement efforts if your organization has many software systems under development.

The real measures are organization-wide, real-world performance against your commercial and non-commercial goals.

## Further reading

- [The 2022 State of DevOps Report](https://cloud.google.com/devops/state-of-devops/)
- [The DevOps Engineer's handbook](https://octopus.com/devops/)
- [White Paper: Measuring Continuous Delivery](https://octopus.com/whitepapers/lv-measuring-continuous-delivery-and-devops)

Happy deployments!