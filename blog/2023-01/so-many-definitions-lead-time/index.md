---
title: Why are there so many definitions of lead time?
description: Find out why there are multiple definitions of lead time and how you can use them to improve software delivery.
author: steve.fenton@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 
isFeatured: false
tags: 
  - DevOps
---

When someone mentions lead times in software delivery, they might mean the definition of lead time from Lead Software Development, the one from DevOps, or something else entirely. Why are there so many definitions of lead time and how do you put them to use?

## Lead time definitions

The [DevOps definition of lead time](https://octopus.com/devops/metrics/) is the time between a change being committed into version control, and the change being deployed to the production environment. This definition is more narrow than the Lean definition of lead time.

In Lean Software Development, created by Mary and Tom Poppendieck based on the Lean manufacturing movement, lead time is measured from when a requirement is discovered, and the fulfillment of that requirement. The Lean movement is based on the Toyota Production System, which defined lead time as the time between a customer placing an order and receiving their car.

## Lead time is a customer measurement

What all of these lead times have in common is that they represent a customer measurement. The reason they differ is the customer is different. Toyota was measuring the system from the perspective of a car buyer, the Poppendiecks were measuring the software development system as the business sees it, and DevOps measures the deployment pipeline from the perspective of the developer as the customer.

The key to successful lead time measurement is that it represents how a customer views the elapsed time. If you run a coffee shop, you might measure the time between a customer placing an order and being handed their coffee. However, this doesn't capture the lead time of the whole system, which should start from when the customer joins the queue. It is important to capture the customer's perception of time.

Passive waiting time should always be included in lead time measurements, but you may be able to omit active waiting time. A famous example of this was the problem faced by [Houston Airport](https://www.theguardian.com/lifeandstyle/2018/sep/07/how-to-beat-bottlenecks-oliver-burkeman), where passengers complained about the wait times for their luggage. The airport hired additional staff and managed to reduce the wait to 8 minutes, but this didn't stop complaints.

Upon discovering that passengers were walking for 1 minute, and waiting for 7 minutes, the solution was to move baggage reclaim to increase the walking time. In general, passengers preferred a longer walk with a shorter wait - even though the luggage was still 8 minutes away from disembarkment.

Despite the success of shifting to active waiting, some passengers may have been negatively impacted by having to travel further to reclaim their luggage. The complaints of minority groups are often lost in averages.

## Cycle times

When you measure the same system from a perspective other than the customer, you are collecting the cycle time. For example, the cycle time for software delivery starts when a work item is picked up and closes when the work is complete. In the car industry, it would time the car from the start to the end of the production line.

Customers don't care about cycle times. If you were waiting for months for your car to be delivered, you would not be consoled by how quickly the factory put it together. You might worry about where the finished car has been sitting for the past six months.

## All measurements are useful

Lead time is valuable because it represents the customer's perception. Identifying your customer and tracking lead times as they see them will ensure any improvements you make impact their experience. Any improvements that speed up a part of your process without reducing lead times is a signal you've optimized the wrong part of the system.

The Theory of Constraints, created by Eli Goldratt, tells us that there is only one constraint in the system. Optimizing anywhere other than the constraint will fail to improve the performance of the whole system.

However, while lead time is vital to ensure your improvement efforts are worthwhile, you may need other measurements to help you find the constraint. Cycle times and other part-system timers will help you work out where optimization is likely to reduce the overall lead time.

...

Happy deployments!