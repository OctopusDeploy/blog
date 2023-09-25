---
title: The 2023 DevOps performance clusters
description: 
author: steve.fenton@octopus.com
visibility: private
published: 2023-10-03
metaImage: 
bannerImage: 
bannerImageAlt: 
isFeatured: false
tags: 
  - DevOps
---

The 2023 Accelerate State of DevOps Report has arrived once more and we're excited to be sponsors once again.

You can read the full report on the [DORA website](https://dora.dev/research/2023/dora-report/).



Throughput and stability

This is how the performance clusters landed in 2022:

| 2022 Performance level | Lead time         | Deployment frequency   | Change failure rate | Mean time to resolve |
|------------------------|-------------------|------------------------|---------------------|----------------------|
| Low                    | 1-6 months        | Monthly to biannually  | 46-60%              | 1 week - 1 month     |
| Medium                 | Weekly to monthly | Weekly to monthly      | 16-30%              | 1 day - 1 week       |
| High                   | 1 day - 1 week    | Multiple times per day | 0-15%               | < 1 day              |

[last year](https://octopus.com/blog/new-devops-performance-clusters)




## Changes to stability metrics

The stability metrics have been adjusted in the 2023 report.

In previous years, respondents could choose from 6 ranges of change failure rates, for example "0-15%". This year, the research team allowed free choice of any value between 0% and 100%. This means the change failure rate data has a higher fidelity than ever.

Perhaps more dramatically, mean time to resolve has been replaced. 

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

Conclusion

- This does point to a link between experience and performance

- If you are similar to a cluster on several measures, but worse on one, you have a great place to start your continuous delivery efforts

- Deployment automation is a great way to improve several DORA metrics and also enables more feedback loops to super-charge your other improvement experiments

Happy deployments!








[specific definitions of lead time](https://octopus.com/blog/definitions-of-lead-time)

[common mistakes in DevOps metrics](https://octopus.com/blog/common-mistakes-devops-metrics)

[DevOps metrics](https://octopus.com/devops/metrics/)

[DORA metrics](https://octopus.com/devops/metrics/dora-metrics/)

[Workplace cultures](https://octopus.com/devops/culture/workplace-topologies/)