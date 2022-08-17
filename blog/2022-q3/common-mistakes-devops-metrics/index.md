---
title: Common mistakes in DevOps metrics
description: TBC
author: steve.fenton@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: blogimage-calculatingdorametrics-2022.webp
bannerImage: blogimage-calculatingdorametrics-2022.webp
bannerImageAlt: A magnifying glass highlights a chart on a dashboard shown on a laptop screen
isFeatured: false
tags:
  - DevOps
---

Metrics are crucial to the process of continuous improvement. However, a balance must be struck between making information visible and being blinded by a flurry of data. You'll need to decide what data to collect as well as what smaller set of data you pay attention to at any time.

If your car had a dashboard that displayed every metric it collects through the engine management system, there would be no window left to see the road. Early cars featured an ammeter, which measured the current between the battery and voltage regulator. This was important as it told you the charging system was working. There was no speedometer, because cars could only achieve a top speed of around 35 miles per hour and the suspension was enough to discourage travel at this speed on cobbled streets.

In a modern car, no dashboard space is given to an ammeter (though a battery light will illuminate if there is a problem), but you will find a speedometer. The current dashboard design reflects the difference in cars and also in the broader system they operate in. Roads are generally smoother, suspension systems are far better, and there are more cars on the road. Attitudes towards safety have also changed.

In the same way, the metrics you collect and display will change over time as you respond to differences in team performance and the system they work within.

As you create and evolve your system of measurement, there are some common mistakes you'll want to avoid.

## Ignoring the data

After all the money you spent collecting and visualizing metrics, you'll find that they aren't used. They might be viewed often, but they aren't being used to change and improve how you deliver value to your customers.

## Activity bias

Activity metrics are perhaps the easiest to collect. You can tell how many hours were worked, the number of lines of code added and removed from version control, or the number of tickets completed. The problem is, not all activity represents progress.

You have to balance activity metrics with outcome-based measurements, which are harder to obtain. Outcome-based metrics are the only well to tell if all the motion is getting you the results you want.

## Measuring too much at once

This can happen gradually, or all at once. You should try to keep your set of current metrics lean and relevant. This means removing metrics from your dashboard when they are no longer useful, even though you enjoy seeing the data.

In some cases, you might decide to leave an automated alert in place to tell you if things are deteriorating, but your main view over metrics must be kept trim.

## Jumping into tools

Data visualization tools like Microsoft Power BI, Tableau, or Google Data Studio are among the coolest software products you'll ever have in your organization. Unlike the many business tools that have table-based and text-based interfaces, data tools have colorful animated charts.

It is easy to be distracted by the task of creating a really exciting dashboard. If you don't start with metric design, you end up with lots of pleasing dashboards that don't impact your daily work. You'll definitely need these tools as they will help you explain reality with a compelling story, but design a lean and useful set of metrics first.

It is better to start in low-fidelity to collect meaningful metrics. You can use a whiteboard or a spreadsheet with manually entered data to test the data. Once you work out which measurements are useful to your organization right now, you can approach the task of automated collection and slick displays.

If you spend a great deal of time creating a stunning dashboard, you might be inadvertently making your metrics more permanent. If you fall in love with your dashboard, you won't want to change it when you need to remove a metric or measure something else.

## Standardization

You might have a development team in your organization who are an example of high-performance in software delivery. They are likely to be tracking a set of metrics that they use as part of their continuous improvement efforts.

It is tempting to take their metrics as a template, standardize it, and roll it out to all teams. The problem with this is that metrics useful in high-performing teams are often different to metrics useful to teams earlier in their improvement efforts.

The skill of metrics is working out what behaviors need to change, and designing a way of measuring the change.

Imagine you were helping a team who had longer cycle times. You've done some research by following some work items to completion and found that the primary cause of delay is in the review stage. Having completed the change, a developer waits 8 hours before someone approves their pull requests.

In theory, if pull requests were completed in minutes, rather than hours, work would flow faster through the value stream. Developers would also be able to complete the work they started, rather than starting new work while they wait for the approval. This requires less switching between different work, which avoids repeating the ramp-up time needed for knowledge work.

To test this theory, you need to collect the cycle time for pull requests from the time they are opened to the time the code is merged into the main branch. You also need to collect the total lead time for the change. You can now test your theory to see if faster pull request approvals reduce the overall lead time.

The metrics also signal to the team what is important *at the current time*.

## Rewarding performance

If a team is working to increase their deployment rate, it can be tempting to incentivize them by telling them they will get a reward if they attain daily deployments. This approach to rewards leads to poor outcomes. A team might let other important work slip to achieve the goal, not to cheat the system but because you've set up a system where you've accidentally made daily deployments more important than anything else.

In the landmark book *Punished By Rewards*, Alfie Kohn explains that attempting to manage using incentives leads to long term harm for your organization. It was found across hundreds of studies that people do worse work when they are offered rewards. Outside of education and work, this practice would be called a *bribe*.

Using metrics to create a competitive atmosphere, either for individual performance, comparison of different teams, or to gamify the workplace (where you introduce game elements as a form of "fun" competition) all lead to trouble. Competition conflicts with what you really need - collaboration.

## Summary

The 5 DORA metrics (there used to be 4, but there's a new one) and the SPACE framework provide pre-built balanced ways to measure software delivery performance. The best set of measurements mixes leading measures of activity and output with lagging measures that indicate whether the organization is achieving it's goals. We cover this in detail in our white paper on [measuring Continuous Delivery](https://octopus.com/resource-center).

Whatever you measure, you need to constantly refine the metrics to ensure they are useful to your team and organization. Ideally, the metrics you collect are relevant to a specific theory you have, like the example of pull request delays causing long lead times.

If you use metrics well, you'll certainly amplify performance and learning as you seek to become one of the elite performers in software delivery.

Happy deployments!

## Further reading

- Punished by Rewards - Alfie Kohn. 1993.
- We have a white paper on [Measuring Continuous Delivery](https://octopus.com/resource-center), which describes different types of metrics and frameworks for measuring DevOps and Continuous Delivery
- You can [find out more on DevOps and Continuous Delivery in our DevOps Engineer's Handbook](https://preprod.octopus.com/devops/)