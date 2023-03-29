---
title: How to measure DevOps mean time to recovery
description: Find out why mean time to recovery has problems and what to do about it.
author: steve.fenton@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 
isFeatured: false
tags: 
  - DevOps
  - DORA Metrics
---

It is very easy to overthink your DevOps metrics. If you've read the amazing and rigorous research undertaken by [DORA](https://www.devops-research.com/research.html), you might worry that you don't have the scientific and statistics knowledge to do the same for your team.

The good news is, you don't have to. The metrics you collect in your team aren't intended for peer-reviewed publication in a journal. They just need to be good enough to assist your continuous improvement process.

One metric that is currently the subject of broader discussion is mean time to recovery (MTTR). In this article you'll see how MTTR is valid for research, the arguments against it, and what you can use to measure your software delivery performance.

## What is MTTR

Mean time to restore, or mean time to recovery, is one of the [DORA Metrics](https://octopus.com/devops/metrics/dora-metrics/), which predict software delivery performance and several organizational outcomes. If you perform well against all the metrics, you'll have working software sooner, happier employees, and a competitive advantage in your industry.

To measure MTTR, you need to collect the duration of each incident from when it started, to when it ended. You would then sort all the incidents by duration and pick the middle one to obtain the *median* incident duration. We'll talk about the shortfalls of averaging incident duration for your team shortly.

:::hint
To calculate an average, you'd sum all the durations and divide the total by the number of incidents. This is the *mean* average.

DORA uses the *median* average in their research. To calculate this, you'd arrange all incident durations from smallest to largest, then pick the value from the middle of the list. If the list has an even number of entries, you'd average the two middle values.
:::

### Why MTTR is useful for industry research

When you are researching performance across many organizations and industries, you can't ask people to provide a list of all incidents for analysis. This would deter many people from responding to the research survey. That means you need to ask them to summarize their experience by providing a comparable number.

The [DORA quick check](https://www.devops-research.com/quickcheck.html) phrases the MTTR question like this:

*For the primary application or service you work on, how long does it generally take to restore service when a service incident or a defect that impacts users occurs (for example, unplanned outage, service impairment)?*

- More than six months
- One to six months
- One week to one month
- One day to one week
- Less than one day
- Less than one hour

Most people working in software delivery have a feel for the typical duration of incidents, especially with the broad buckets used in the answer. You can probably answer this for your team from memory with a reasonable level of accuracy.

The researchers can use this information to find [performance groups](https://octopus.com/blog/new-devops-performance-clusters) in the data. They can also look for relationships between various practices and their impact on business outcomes. The DevOps structural equation model is built using these findings.

### The reason MTTR isn't right for your team

While MTTR is useful in research for comparisons and clustering, this isn't how you'll use incident information in your organization. Your main use of incident duration is to learn from service outages and improve how you handle them in the future.

For continuous improvement purposes, using an average hides important signals. You need more fine-grained information to understand how well you handle faults and to find their causes.

The Verica Open Incident Database (VOID) has more than 10,000 incidents shared by almost 600 organizations. In [the VOID report](https://www.thevoid.community/report), they provide an analysis of the incidents. The 2022 report made the following comment about MTTR:

> MTTR isn’t a viable metric for the reliability of complex software systems for a myriad of reasons, notably due to its underlying variance.

Within your organization, MTTR will only be a useful measure if you have thousands of incidents each month. Most teams will be hoping for a smaller number of incidents. With fewer incidents, the use of averages results in a highly volatile metric, which can go up even when incident management improves.

Averages are most useful when you have a normal distribution. Incident durations skew heavily to the left of the chart as most incidents can be solved quickly. The VOID database found most incidents are resolved in under 2 hours. There is then a long thin tail of values that can cause wild variation in mean and median averages.

You could eliminate this variation by excluding outliers, but then you'd be hiding valuable information. You need a better way to use this data to improve your process.

### Where restore times remain useful

Instead of zipping up your incidents into an average, plot each duration on a chart. You can use a scatter plot or a box-and-whisker chart to visualize durations without losing fidelity. This will show you trends and outliers, which is more useful than an average.

![A scatter plot showing incident durations](time-to-restore-scatter.png)

You can now understand the trend in resoltion times to see if you are improving over time. You can also identify outliers that took longer to solve and have conversations about how these could be handled better. Resolution times remain useful as part of your journey into exploring incidents and improving incident management and system stability.

If incidents require a code fix, the restore time will reflect the performance of your deployment pipeline. Being able to quickly and safely deploy new versions of your software has positive effects beyond incident management. The restore times also encourage the introduction of monitoring and alerting tools, which significantly improve your ability to detect problems before a customer is impacted.

To get the most out of incident duration data, make sure you have consistent definitions for:

- What is an incident
- What the start time is
- What the end time is

Should you classify something as an incident if a system failed, but didn't impact customers. For example, your instant-search feature may have stopped working, but the fallback of letting users round-trip the search form to get a response still worked - is this an incident? It's quite subjective. What if your web server crashed, but your edge cache continued to serve requests?

The exact start time and end time of an incident can be tricky to pin down. Do you start the clock when the CPUs get hot, when the response times slow down, or when the first customer gets an error? The same goes for the end time - is it when customers can use the system again, or when you've mitigated the issue, finished fixing a bug, or when you've put in a permanent fix to prevent a similar incident in the future?

By creating a strong definition for indicents and their measurement, you can make sure your data is useful in a comparison.

You will likely reach a point where time to restore no longer offers you the prompts needed to find the next level of improvements. When you can easily deploy changes and receive early warnings of faults, you'll need some new measurement ideas. This is where the SPACE framework can help you measure your incident response capability.

## Using the SPACE Framework to measure incident response

[The SPACE framework](https://octopus.com/devops/metrics/space-framework/)

Satisfaction and wellbeing

Are people managing incidents happy with:

- The incident management process
- The on-call schedule
- How easy it is to escalate or access specialists to help during an incident
- What local times did pagers go off

Performance

- System reliability
- Time between incident conditions and awareness of the incident

The time to restore service would fit into this category.

Activity

Things you can count

- Number of alerts raised by monitoring tools
- Number of incidents raised
- Number of concurrent incidents

Communication and collaboration

- Number of people involved per incident
- How many different teams were involved in an incident
- Number of chat channels opened for an incident
- How many times the incident report was viewed (or given a positive rating, or referenced in other incidents)

Efficieny and flow

- Number of times an incident is reassigned
- Number of attempts before a successful mitigation



These measurements are useful as they highlight the hidden costs of managing incidents and provide concrete areas that could be improved.

And while you're at it... ask your customers how they rate your reliability!


These will have more direct impact on your incident response process than time to recover.

## Transitioning from MTTR

Introduce time to restore scatter charts alongside any existing metrics you have, including MTTR. The idea is to familiarize the new measurements before you retire the old ones.

Once the new concepts become established, you can de-prioritize MTTR and shift the focus onto the more useful metrics.

The same process can help you normalize the process of changing metrics over time, for example, if you expand from [DORA metrics](https://octopus.com/devops/metrics/dora-metrics/) to [the SPACE Framework](https://octopus.com/devops/metrics/space-framework/).

## Beyond the numbers

Metrics are useful because they prevent you from fooling yourself with convincing narratives. If you work without any measurements, you'll explain away incidents that take too long to resolve. It was an exceptional case. It could never happen again. The numbers will quickly dispel the illusions we sometimes create, as they tell you that the incident isn't unusual and similar things happen repeatedly.

What you really must do is run retrospectives for each incident. The numbers will contribute to the information you have to come up with improvement ideas. Running the retrospective as soon as the incident is resolved ensures vital context isn't lost as people move on to other things and forget the details.

It is more important to learn from incidents than it is to achieve some arbitrary goals around time to recover.

The numbers don't drive continuous improvement, they just remind you of the reality so you can apply some human ingenuity and make things better each week, forever. Use the numbers to identify and remove bias and logical fallacy from your discussions, so you can deal with the reality that is before you.

## Incident fallacies

The most common fallacies preventing teams from improving their software delivery performance are...

https://en.wikipedia.org/wiki/List_of_fallacies

- Appeal to probability
- Questionable cause
- Relevance fallacy

### Appeal to probability

- Appeal to probability: Assuming you know the cause of an incident because your idea seems highly likely. You need to work out how to test your theory.

### Questionable cause

When two things happen a similar times it is easy to draw false conclusions either about the presence of a relationship, or the direction of that relationship. For example, you might believe your incident was caused by a high number of concurrent requests on your load balancer, which overloaded your web servers. If you dig deeper, you are likely to find the concurrent requests are a symptom of the load balancer getting slower responses from the web server (possibly caused by the database being overloaded, which left the web server waiting a long time for data).

You should build an incident dashboard within your monitoring tool with metrics from all suspect components. Good monitoring tools will help you identify the precise order of the failure cascade. If you have the database load, web server response times, and load balancer queues visualized on time series charts, you can quickly see which one faulted first and prove that the load balancer queue didn't occur until after the problem started (or, indeed, that it *did* fail first as we can change our mind in the face of evidence).

### Relevance fallacy

Highly relevant to incident retrospectives, relevance fallacies lead you to dismiss possibilities because they sound outlandish or incredible. 

Set a clear divergent stage of the retrospective where all ideas are accepted without question and added to the list of possibilities. Ask everyone to hold onto their thoughts and judgments at this stage as you just want all ideas. Once you have exhausted the suggestions, you can pick one or two possibilities and work out how they might be eliminated. You can then test them to see if they remain possible or should be discarded. Repeat this until you narrow down the items.

## Use incident reviews, not root cause analysis

Be cautious about root cause analysis as software systems rarely have a single root cause, it's normally a combination of several contributing factors. Root cause analysis typically focuses on the people closest to the incident not on the broader systemic issues. Aim instead for a post-incident review that details all the things that were happening and what was done to mitigate and solve it. Your incident reviews can contribute directly to a reduction in resolution times as they capture the learning and make it available to people handling the next incident.

> Safety is the capability to absorb an incident, not the absence of failure, and incident reviews are blameless learning opportunities. - Adrian Cockcroft

The cause of an incident is never a person, it's a whole system within which people work. If someone logged onto a server and accidentally selected "shut down" instead of "log out", that's a fault in the system. Why hasn't the "shut down" option been hidden? Why do they even need to access servers directly - couldn't this be done with a runbook?

You can use past incident reports for analysis and to run review workshops where you discuss the incident. This spreads knowledge and generates ideas that can improve your current process and change future decisions.


The previous incident reporting format Microsoft used was:

- Summary of impact
- Preliminary root cause
- Mitigation
- Next steps

The post-incident review format Microsoft uses for incidents is:

- What happened?
- What went wrong, and why?
- How did we respond?
- How are we making incidents like this less likely 
or less impactful?
- How can our customers and partners make 
incidents like this less impactful

## Conclusion


Treat incidents as opportunities to learn
Favor in-depth analysis over shallow metrics
Treat humans as solutions, not problems
Study what goes Right Along with what goes wrong

## Further reading

- [DevOps metrics](https://octopus.com/devops/metrics/)
- [Measuring Continuous Delivery and DevOps white paper](https://octopus.com/whitepapers/lv-measuring-continuous-delivery-and-devops)
- [DORA DevOps research](https://www.devops-research.com/research.html)
- [The Verica Open Incident Database (VOID)](https://www.thevoid.community/)

Happy deployments!