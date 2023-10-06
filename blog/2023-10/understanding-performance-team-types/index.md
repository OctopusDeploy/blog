---
title: Understanding performance through team types
description: Find out how the descriptive team types in the State of DevOps report can help you improve.
author: steve.fenton@octopus.com
visibility: private
published: 2023-10-10-1400
metaImage: blogimage-meantimerecovermetric-2023.png
bannerImage: blogimage-meantimerecovermetric-2023.png
bannerImageAlt: Person crouching ready to start a sprint on top of a giant stop watch.
isFeatured: false
tags: 
  - DevOps
---

Thanks to the research team at DORA, we can access a [model of technical and cultural capabilities](https://dora.dev/research/) to direct our continuous improvement activity.

The goal isn't to adopt every item like a checklist; it's [not a maturity model](https://octopus.com/blog/devops-uses-capability-not-maturity). Instead, you can use it as a map to find practices and techniques to help you improve based on your current circumstances.

For example, if you find it hard to create automated integration tests because the application state causes failures, you might benefit from the *test data management* capability.

The [2023 State of DevOps Report](https://dora.dev/research/2023/) provides another way to examine and improve your performance. The report has 4 descriptive team types based on a combination of 3 performance measures:

- Software delivery performance
- Operational performance
- User-centricity

These team types are helpful as they provide a way to identify with different characteristics and understand their strengths and weaknesses.

## Why clusters are useful

Clustering has been a feature of the State of DevOps report for a long time. The software delivery performance clusters group teams based on throughput and stability. This usually results in low, medium, high, and elite performance clusters.

Although clustering is a statistical exercise, it provides personas you can use to think about how you deliver your software. You can assess your performance by comparing your metrics to the clusters in the reports.

Knowing there are safety-critical and regulated industries represented in the elite cluster also inspires you to strive for this level of performance. The culture and practices in the DORA model aren't compromises to these concerns; they make you safer and more compliant.

The 2022 report introduced [descriptive clusters](https://octopus.com/blog/new-devops-performance-clusters#the-sdo-performance-clusters). Instead of providing a continuum of performance levels, the descriptive groups provided situational examples based on software delivery and operational performance.

Instead of aiming for elite performance, the descriptive clusters let you assess the needs of the software and match it with an appropriate performance level. For example, a team searching for product/market fit might sacrifice reliability to increase their rate of experimentation.

You can use performance and descriptive clusters as lenses through which you can critique your practices.

## The new descriptive team types

With the addition of user-centricity, 4 team types emerge:

- User-centric
- Feature-driven
- Developing
- Balanced

The researchers found the team types by splitting performance using the assessment criteria (software delivery and operational performance, and user-centricity). But the team types also predict certain outcomes. You can use these team types as personas to work out how to increase the impact of software delivery and improve developer experience.

![The relative performance of the four team types against the three performance measures](team-type-performance.png)

You won't match to a single team type, and you'll move around as you adjust your process. Just pick the nearest example and see what ideas emerge.

Let's look at each of the team types in more detail.

### User-centric

User-centric teams have strong software delivery and operations performance. Their focus on user needs leads to the highest levels of organizational performance. These teams have worked out how to unlock the impact of software on organizational goals.

![The chart shows user-centric teams have high performance against all measures, but with higher burnout than balanced teams.](user-centric-outcomes.png)

If you're a user-centric team, you must watch for burnout. It's possible for things to feel intense, even though job satisfaction is high. Removing toil with automation can reduce burnout. 

### Feature-driven

Feature-driven teams have incredible software delivery performance but they're disconnected from users. This can happen if there's no mechanism to obtain user feedback or if this feedback never makes it back to the developers. This means the software has less impact on organizational performance. These teams have high levels of burnout and lack job satisfaction.

![The chart shows feature-driven teams have strong team performance that doesn't convert into organizational performance. There is high burnout and low job satisfaction.](feature-driven-outcomes.png)

You need to reconnect with your users if you're a feature-driven team. Delivering frequently should provide an opportunity for valuable feedback loops. Find ways to bring user feedback into the planning process to increase the value of the features delivered.

### Developing

Often found in smaller organizations, developing teams are building towards one of the other team types. These teams are searching for features to make their product attractive to customers while building their skills. Despite high job satisfaction, these teams are the most prone to burnout.

![The chart shows developing teams have strong outcomes with the highest level of burnout.](developing-outcomes.png)

If you're in a developing team, try to replace heavyweight processes with automation. This will improve your operational performance and result in less unplanned work and interruptions. Introducing technical practices will help maintain momentum for the long haul, rather than depending on people to fill gaps.

### Balanced

Balanced teams have worked out how to achieve strong performance with low burnout. These teams have skills across many technical practices and cultural capabilities. They have a good impact on organizational outcomes but can still improve.

![The chart shows balanced teams achieve good outcomes with the lowest levels of burnout.](balanced-outcomes.png)

If you're in a balanced team, you can increase performance at the organizational level by being more user-centric. As you look to incorporate feedback loops, you must ensure you set a sustainable pace to avoid burnout.

## The best team type

There's no *best* team type. That's the beauty of descriptive types over performance clusters. Your team type will change over time based on the product's needs and your team's skills.

Each team type has at least one adjustment they could make that will optimize one or more outcomes. Because each team type has at least one weakness, you can use them to avoid over-optimizing for a single measurement.

For example, the user-centric team has excellent performance but higher burnout, so it has something to learn from a balanced team. Similarly, the balanced team is making less impact on the organization, so it can learn from developing and user-centric team types.

After you assess your team type, you can work out your desired state and adjust accordingly.

![Relative predicted outcomes for the four team types](team-type-outcomes.png)

Watch for burnout in these visualizations. Unlike the other measures, a lower value is better. The challenge is to find ways to increase the other outcomes while reducing burnout.

## Flat-out speed isn't the goal

There has been a problem with *speed* in the Agile community. There may be a useful distinction between the terms *faster* and *sooner*.

If we work in small batches and release changes to users regularly, we can get feedback sooner. The feedback lets us change what we do next. If we delight users with a feature, we can stop working on it and pivot to a new area. This maximizes the amount of work not done.

If you aim instead to deliver faster, you ignore feedback as it slows down your software delivery. This is the problem with flat-out speed - like refusing to pull over to check a map because it will slow you down.

:::hint
A study of [feature experimentation at Microsoft](https://ai.stanford.edu/~ronnyk/ExPThinkWeek2009Public.pdf) found that without a functioning feedback loop, 60%-90% of your ideas won't improve the metric you intended to improve.
:::

Successful teams aim to release sooner, not faster. This is the crucial difference between user-centric and feature-driven teams. The user-centric teams stop and check directions, while feature-driven teams point in a direction and stop for nobody.

Product development is a process of removing uncertainty through validated learning. You need to assess the impact of the software on its users each time you release a change. You may deliver fewer features, but they'll each be more valuable.

The most dangerous aspect of flat-out speed is that the codebase becomes more complex, and the team degrades the user experience because they're too focused on being fast. The long-term prospects for feature-driven teams aren't good. They end up damaging the perception of their product and their future performance with low-value features.

The solution to the problem of *fast delivery* of the *wrong thing* is user focus. You can assess your user focus by considering the following:

- How well do you understand the needs of your users?
- Are you set up to meet user needs?
- Do you prioritize user feedback?

## Software delivery success

The DORA metrics of throughput and stability are predictors of organizational success. It's tempting to trust in this relationship when measuring your software delivery performance.

The presence of the feature-driven trap should convince you to validate that your software delivery is translating into meeting and exceeding goals at the organizational level.

Your greatest missed opportunity would be to create a team that can deliver elite software delivery performance without validating their ideas with user feedback.

You can reclaim the time you invest in establishing strong feedback loops when you can stop work on a feature that users love, even though you planned to add more to it. You also reclaim time in the features you choose not to add because they won't impact users.

Happy deployments!