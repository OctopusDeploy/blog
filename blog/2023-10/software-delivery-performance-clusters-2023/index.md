---
title: The 2023 DevOps performance clusters
description: 
author: steve.fenton@octopus.com
visibility: private
published: 2023-10-03
metaImage: blogimage-dorametrics-2023.png
bannerImage: blogimage-dorametrics-2023.png
bannerImageAlt: A slightly transparent computer screen with someone analyzing data behind it, with different metrics floating around the person. 
isFeatured: false
tags: 
  - DevOps
---

The 2023 Accelerate State of DevOps Report has arrived and we're excited to be sponsors once again. The report establishes the relationship software delivery performance has with organizational goals and provides concrete practices that can improve outcomes.

You can read the full report on the [DORA website](https://dora.dev/research/2023/dora-report/).

In this article, you'll find more information about software delivery performance and how it has changed since last year's report.

## Software delivery performance

The traditional view is that there is a trade-off between throughput and stability. If you release changes sooner, and deploy more often, you'd expect to have more failures and lower stability. The evidence provides a counter-intuitive finding. High-performing teams achieve both throughput and stability, while teams who deploy less often have a higher change failure rate and longer restoration times.

Throughput:

- Deployment frequency
- Lead time for changes

Stability:

- Change failure rate
- Mean time to recovery

You can find out more information on these metrics in our overview of [DORA metrics](https://octopus.com/devops/metrics/dora-metrics/).

In the 2022 report, three performance levels emerged from the data:

| 2022 Performance level | Lead time         | Deployment frequency   | Change failure rate | Mean time to resolve |
|------------------------|-------------------|------------------------|---------------------|----------------------|
| Low                    | 1-6 months        | Monthly to biannually  | 46-60%              | 1 week - 1 month     |
| Medium                 | Weekly to monthly | Weekly to monthly      | 16-30%              | 1 day - 1 week       |
| High                   | 1 day - 1 week    | Multiple times per day | 0-15%               | < 1 day              |

With this table, you can see the strong relationship between low throughput (lead time and deployment frequency) and poor stability (change failure rate and time to resolve).

## Changes to stability metrics

To dig deeper into stability, the DORA researchers adjusted the questions this year.

In previous years, respondents could choose from 6 ranges of change failure rates, for example "0-15%". This year, the research team allowed free choice of any value between 0% and 100%. This means the change failure rate data has a higher fidelity than ever.

Perhaps more dramatically, mean time to resolve has been replaced. Over the past year we've had many conversations about this within the [DORA Community](https://dora.community/).

[how you'd use time to recover in your team](https://octopus.com/blog/how-to-measure-mean-time-to-resolve)

- Change failure rate precision

- Deployment failure recovery times replaces MTTR



New clusters (welcome back Team Elite)
- Clusters are useful to describe different performance characteristics

- The best comparison is for the same application over time

- Comparisons between apps, teams, and organizations reflects difference for too many variables


| 2022 Performance level | Lead time            | Deployment frequency           | Change failure rate | Failed deployment recovery time |
|------------------------|----------------------|--------------------------------|---------------------|---------------------------------|
| Low                    | **1 week - 1 month** | **Once a week - once a month** | **64%**             | **1 month - 6 months**          |
| Medium                 | 1 week - 1 month     | Once a week - once a month     | **15%**             | 1 day - 1 week                  |
| High                   | 1 day - 1 week       | **Once a day - once a week**   | 10%                 | < 1 day                         |
| **Elite**              | < 1 day              | On demand                      | 5%                  | < 1 hour                        |

The low performance cluster has shorter lead times and deploys more often, but has a higher change failure rate and longer failed deployment recovery times than last year. 18% of responses.

The medium performance cluster is similar to last year, but with lower change failure rates. 33% of responses.

The high performance cluster is very similar to last year, but deployes less frequently. 31% of responses.

The elite cluster has returned after a year's break. Lead times under a day, deployment made on demand, low change failure rates and fast recovery times. 18% of responses.

The 2022 survey had fewer responses from people with more than 10 years experience. 50% of respondends to the 2023 report had at least 15 years experience. Performance is increased with experience, which is intuitively what we'd think.

## How to use these clusters

With any DevOps metrics, the best comparison you can make is for the same application over time. If you are improving the application's performance against these measures, you're making progress.

There a still some ways these clusters can be used to help with this continuous improvement process.

If your measurements are similar to one of the clusters, but you are under performing on one metric, you now have a good place to start improving. For example, if you fit the *medium* performance cluster, except for having a higher change failure rate, what could you do to improve this? You can use the [DORA Core Model](https://dora.dev/research/) to find capabilities that can help, such as test automation.

You should take an experimental approach to applying these capabilities, introducing or improving a capability and observing its impact on the metrics. This is how you'll find which practices make an impact for your specific application and team.

The other key use of the clusters is in imagining what is possible. There are organizations in many industries represeted in the *elite* cluster, including safety-critical and regulated industries. It is possible to achieve this performance level without making things less safe and without taking shortcuts.

## Conclusion

After last year's report, it seemed likely that there was a link between experience and software delivery performance. The presence of respondents with more than 10 years' experience seems linked to the number of higher-performing teams.



- Deployment automation is a great way to improve several DORA metrics and also enables more feedback loops to super-charge your other improvement experiments

Happy deployments!








[specific definitions of lead time](https://octopus.com/blog/definitions-of-lead-time)

[common mistakes in DevOps metrics](https://octopus.com/blog/common-mistakes-devops-metrics)

[DevOps metrics](https://octopus.com/devops/metrics/)

[Workplace cultures](https://octopus.com/devops/culture/workplace-topologies/)

You can read [why this was different to the 2021 report](https://octopus.com/blog/new-devops-performance-clusters).
