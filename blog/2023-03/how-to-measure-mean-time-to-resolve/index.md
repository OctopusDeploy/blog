---
title: How to measure DevOps mean time to resolve
description: Find out why mean time to resolve has problems and what to do about it.
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

One of the DORA Community discussions last year touched on Mean Time To Recovery (MTTR) and whether it was a valid measure for software delivery performance. MTTR is one of the [DORA Metrics](https://octopus.com/devops/metrics/dora-metrics/) and is used for the performance clusters in the Accelerate State of DevOps report.


The Verica Open Incident Database (VOID) takes inspiration from the aviation industry, where incidents and near-misses are treated as an opportunity to study, learn, and share information. Not all software is safety-critical, but it is playing an ever-increasing role in industries that are, such as transport, infrastructure, and healthcare. The resulting socio-technical systems are complex and may be best understood by analyzing failures.

High-trust and low-blame cultures learn and improve when something goes wrong. LINK TO DEVOPS CULTURE

An annual report based on the incident database of more than 10,000 incidents from almost 600 organizations... questions MTTR.

> MTTR isn’t a viable metric for the reliability of complex software systems for a myriad of reasons, notably due to its underlying variance. - The VOID Report

## What is MTTR

Mean time to recover... the average (median) time to recover from an incident.

You measure from when an incident started, to when it ended, and then convert all your incidents into a median

:::hint
To calculate an average, you'd sum all the durations and divide it by the number of incidents. This is the *mean* average.

DORA uses the *median* average in their research. To calculate this, you'd arrange all incident durations from smallest to largest, then pick the value from the middle of the list. If the list has an even number of entries, you'd average the two middle values.
:::

### Why MTTR is useful for industry research

When you compare organizations, you aren't trying to find the root cause of problems or improve their process - you are creating a benchmark for comparison.

People inside these organizations need to be interested in a more fine-grained level of details.

### The reason MTTR isn't right for your team

MTTR is actually a pretty good metric... if you have a huge number of incidents. When you have a small sample, which really most of us are hoping to achieve, it stops working in a reasonable way. If you really want to use MTTR, you need to aim for perhaps 10,000 incidents a month. This isn't an aspirational goal.

The exact start time and end time of an incident can be tricky to pin down. Do you start the clock when the CPUs get hot, when the response times slow down, or when the first customer gets an error? The same goes for the end time - is it when customers can use the system again, or when you've mitigated the issue, finished fixing a bug, or when you've put in a permanent fix to prevent a similar incident in the future? These factors make it hard to create a reusable definition of incident duration.

Should you classify something as an incident if a system failed, but didn't impact customers. For example, your instant-search feature may have stopped working, but the fallback of letting users round-trip the search form to get a response still worked - is this an incident? It's quite subjective. What if your web server crashed, but your edge cache continued to serve requests?

Incident data doesn't follow a normal distribution, it skews heavily left... this is mostly a good thing because we resolve things quickly a lot of the time (under 2 hours) but it also means a long thin tail of values that can create wild variation, no matter which average you use. If you don't have a normal distribution, you shouldn't use the mean. Median (used by DORA) and mode are also problematic and don't represent the data accurately. Averages have too much variance based on the weight of the short resolutions and the pull of outliers.

Štěpán Davidovič’s - Google study? Ran Monte Carlo simulations to compare a control group with artificially reduced incident durations and found that MTTR increased between 20-40% of the time, despite the durations being artificially reduced.

### Where restore times remain useful

The first thing to throw out is the M in MTTR. Don't track the mean time to resolve, track the time to resolve each incident.

Plot the data as a time series, using either a scatter chart or a box-and-whisker diagram.

As part of your continuous improvement process, you can assess both the trend (whether you are resolving incidents faster or slower over time) and outliers (incidents that took far longer to solve than normal) to kick off conversations about how things could be improved.

Resolution times are the start of your journey into exploring incidents and improving incident response and system stability.

If incidents require a code fix, the restore time will reflect the performance of your deployment pipeline. Being able to quickly and safely deploy new versions of your software has positive effects beyond incident management. The restore times also encourage the introduction of monitoring and alerting tools, which significantly improve your ability to detect problems before a customer is impacted. You may find it useful to use time to restore to drive deployment automation and monitoring, which will improve your software delivery performance dramatically.

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





### Incidents as socio-technical systems MERGE INTO ABOVE

There's a certain pattern that emerges from incident response. The situation is dynamic and uncertain, and the different specialists involved need to quickly and accurately share information and plan their activities. The coordination has a cost, but this is offset by the benefit of having experts on hand who can manage specific parts of an incident. 

Cost of co-ordination (Dr. Laura Maquire) advocates for socio-technical measurement system for incidents.

- How many people involved and from how many teams
- What time of day people were paged
- How many tools were needed
- How many chat channels were opened
- How many concurrent incidents
- How many people view the incident report

These measurements are useful as they highlight the hidden costs of managing incidents and provide concrete areas that could be improved.

And while you're at it... ask your customers how they rate your reliability!


These will have more direct impact on your incident response process than time to recover.

## Transitioning from MTTR

Introduce time to restore scatter charts alongside any existing metrics you have, including MTTR. The idea is to familiarize the new measurements before you retire the old ones.

Once the new concepts become established, you can de-prioritize MTTR and shift the focus onto the more useful metrics.

The same process can help you normalize the process of changing metrics over time, for example if you expand from [DORA metrics](https://octopus.com/devops/metrics/dora-metrics/) to [the SPACE Framework](https://octopus.com/devops/metrics/space-framework/).

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

- [DevOps engineer's handbook](https://octopus.com/devops/)
- [Measuring Continuous Delivery and DevOps](https://octopus.com/whitepapers/lv-measuring-continuous-delivery-and-devops)
- [The Verica Open Incident Database (VOID)](https://www.thevoid.community/)

Happy deployments!