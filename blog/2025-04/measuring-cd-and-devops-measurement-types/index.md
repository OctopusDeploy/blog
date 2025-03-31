---
title: The 3 measurement types from Measuring Continuous Delivery and DevOps
description: Find out about 3 general types of measurement that will help you better understand CD and DevOps metrics.
author: steve.fenton@octopus.com
visibility: public
published: 2025-04-02-1400
metaImage: blogimage-bestpracticecicd-2022.png
bannerImage: blogimage-bestpracticecicd-2022.png
bannerImageAlt: Person with a checklist stands in front of DevOps infinity symbol.
isFeatured: false
tags: 
  - DevOps
  - Continuous Delivery
---

Your continuous improvement process sits neatly between a system of measurements. You can use metrics to generate focus areas where you need to improve and use the metrics again to track your progress.

The perfect metrics would be real-world organization outcomes that update in real time as you try adjustments. The reality is that perfect metrics don't exist. The outcomes that are crucial to your organization tend to move slowly and imperceptibly in response to an improvement. Moving from monthly to weekly deployments won't immediately bring a rush of new customers, but delivering small batches regularly does help improve outcomes like this in the long run.

In the absence of perfect metrics, we can still create a helpful measurement system by finding small responsive numbers that predict changes for the big-ticket items. Industry research can help us with this, as many relationships have been found between software delivery and organizational success.

Our white paper, [Measuring Continuous Delivery and DevOps](https://octopus.com/whitepapers/measuring-continuous-delivery-and-devops), examines metric design and provides ways to power continuous improvement with numbers and techniques like statement-based assessments.

This post summarizes 3 types of measurement from the white paper:

1. Leading versus lagging indicators  
2. Instrumented versus perceptual data  
3. System versus survey data

## 1. Leading versus lagging indicators

Leading indicators help you predict future performance. You can use their early signals to predict changes in performance and take action to adjust the future outcome.

Lagging indicators tell you what happened in the past, but rather than being predictive, they're typically factual.

When you monitor a software system, leading indicators help you take action before a system becomes unavailable. For example, tracking CPU use lets us see when a feature uses too many resources and puts other features at risk. This lets you scale your resources or switch off the feature before the problem affects customers.

If you wait for a lagging indicator, like screen loading times, your users will notice the problem before you do. That doesn't mean you shouldn't measure lagging indicators, as these show the *system's real performance*. Leading indicators can alert you to a potential problem earlier, but they may change for reasons you don't expect.

This explains the importance of using leading and lagging indicators to measure software delivery capability. The leading indicators help you react earlier, but the lagging indicators tell you the actual state of the whole value stream.

The most important measurements to your organization are likely to be lagging indicators. For example, your organization cares about *revenue*. This measurement tells you about money that has (or hasn't) come into the business. A predictive indicator, like net promoter score (NPS), can help you spot problems that might affect future revenue, but the money that turns up is a cold, hard fact.

Leading indicators are often imperfect. We accept this because we value the ability to take action sooner. You should combine leading and lagging indicators to measure Continuous Delivery, DevOps, and software delivery performance.

## 2. Instrumented versus perceptual data

Instrumented data is a measurement you can collect automatically, like temperature. Perceptual data covers the human experience, like if the temperature is comfortable for you.  
Instrumented data is popular because it's easy to get. However, perceptual data can capture complex relationships across many factors.

When you ask someone if a room temperature is comfortable, their answer goes beyond measurable temperature. The answer could include factors like:

- Humidity  
- Outdoor conditions  
- Their activity  
- Clothing  
- Personal preference

If you ask employees if they feel energized or burned out, they report feeling burned out before their productivity drops.

Most organizations are good at collecting instrumented data. You should become just as good at collecting perceptual data as it often provides uniquely nuanced insights.

## 3. System versus survey data

If a team uses an internal tracking tool for features and a customer-facing tool to capture bugs, for example, you must collect data from both systems. If you only collect metrics from the internal tracking system, this work will get prioritized and optimized at the cost of customer-reported issues.

You must also ensure teams use tools as expected. In factories with time clocks, it was common for the first worker to clock their colleagues in to stop them from getting in trouble. According to the clock, worked hours would be far more than the actual hours, and any decisions based on this data were likely wrong.

In modern work tracking systems, users can mark work as complete before deploying it to production, which can lead to inaccurate lead times.

Survey data needs more effort, but it can capture a richness of information not available from a system. You can collect perceptual data from surveys and instrumented data you can't get from a system. For example, if you haven't installed tools to support deployment automation, you could use a survey to ask how often a team deploys to production.

When designing your surveys, consider the frequency and effort needed to respond. It helps if you coordinate your efforts to avoid flooding people with surveys from different parts of the organization.

You may need some statistical skills to account for bias and help establish significant relationships in the data.

For both system and survey data, if the organization isn't a safe space for honest feedback, the data will be what people believe to be the 'correct answer'.

People can manipulate system data as easily as survey data. Marking work as complete before it becomes overdue and then tracking the work outside the system to finish it, for example.  

You can't measure a complex system (like the software delivery process) on system data alone. You should use both system and survey data in your measurement approach. Use the natural conflicts to discover and fix weaknesses in your measurement approach.

## It's all about improvement

The techniques in the white paper, [Measuring Continuous Delivery and DevOps](https://octopus.com/whitepapers/measuring-continuous-delivery-and-devops), can help reveal improvement opportunities. They provide a mechanism to design your improvements deliberately and scientifically. If you make a change to increase deployment rates but instead deploy less, you know the change didn't have the intended effect and can try something else.

Over time, you need to respond to weaker signals rather than believing there are no further improvements to make. Teams and organizations that continuously improve will outperform those that think they're done.

You can [download the free white paper](https://octopus.com/whitepapers/measuring-continuous-delivery-and-devops) to learn more.  

Happy deployments!