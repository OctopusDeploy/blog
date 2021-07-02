---
title: Safe Schema Updates - Loose Coupling Mitigates Human Problems
description: On the human challenges associated with monolithic architectures, and how looser coupling mitigates these challenges.
author: alex.yates@dlmconsultants.com
visibility: private
published: 2022-08-18-1400
tags:
 - DevOps
 - Database Deployments
 - Deployment Patterns
---

This blog post is part 5 of my Safe Schema Updates series. The other posts in this series are available at the following links:

!include <safe-schema-updates-posts>

> *“No matter what the problem is, it’s always a people problem.”*
> *Gerald Weinberg*

In the previous post (Part 4) we focussed on technical issues associated with software/database architecture. In this post we’ll focus on the human issues.

[Conway’s Law](https://en.wikipedia.org/wiki/Conway%27s_law) dictates that organisations are limited to design systems that mirror their internal communication patterns. As Eric S Raymond so eloquently put it in [The New Hacker’s Dictionary](https://www.goodreads.com/book/show/104746.The_New_Hacker_s_Dictionary), *“if you have four groups working on a compiler, you'll get a 4-pass compiler”*. 

Our team structures measurably affect our software architectures… but the reverse can also be true: complicated architectures can foster painful and toxic bureaucracy and working cultures. Together, left unchecked, it's possible for them to form a vicious circle. 

When things go wrong with critical but complicated monoliths, and when it’s difficult to attribute cause and effect, individuals are generally quick to point fingers and cover their own backs. If we are honest with ourselves, we can probably all think of times when we’ve acted defensively, even if we didn’t know whether we were at fault. We are all valuable but imperfect humans, and we share a deep personal instinct for self-preservation.

Patrick Lencioni talks about conflict, commitment, and accountability in [The Five Dysfunctions of a Team](https://octopus.com/blog/devops-reading-list#dysfunc). His fourth dysfunction explicitly calls out the avoidance of accountability as being detrimental to effective teamwork:
 
![The Five Dysfunctions of a Team](five-disfunctions.jpg)

*Image source: [https://medium.com/taskworld-blog/lencionis-5-dysfunctions-of-a-team-330d58b2cd81](https://medium.com/taskworld-blog/lencionis-5-dysfunctions-of-a-team-330d58b2cd81)*

In part 2 of this series, as part of our discussion into reliability vs robustness, we discussed Richard Cook’s insightful research into [how complex systems fail](https://how.complexsystems.fail/). In particular, we looked at the problems with traditional views about accountability or blame and “root cause analysis” within complex systems, where failures are often caused by many, seemingly unrelated factors. These challenges are inescapable, but we can help ourselves by designing systems that are easier to observe and where relationships, dependencies and expectations are more clearly defined.

When we break our complicated, monolithic architectures up into sets of smaller, more loosely coupled systems, it helps everyone to understand the relationship between cause and effect. Through proper instrumentation/telemetry/monitoring we are able to track various performance metrics for each service. When systems fail, there’s less confusion, arguing, finger pointing and politics. It’s much easier to focus on understanding, fixing and reviewing failures - and to make changes to avoid issues recurring in the future.

It’s much easier to learn and improve. Everyone benefits.

## Service Level Objectives (SLOs) and Downtime Budgets

The DevOps movement rejects the idea that development teams should be targeted on the speed of delivery, while separate operations teams are targeted on stability. The conflict between the objectives of these teams does not incentivise cooperation or systems thinking.

By breaking up monoliths into separate services, it allows for smaller, cross-functional teams to take responsibility for both the speed of delivery *and* the stability of their own services. Where there are conflicts between speed/stability objectives, they can be weighed against each other by the people who best understand the customer needs and technical details and who jointly own both the speed and stability objectives.

To support this, each service could be given specific set of public Service Level Objectives (SLOs). Downstream services will be aware of these SLOs and can design their own services defensively with these SLOs in mind. For example, if the sales service (from the previous post in this series) promises 99.9% uptime, the support service will not be designed to expect 99.99%.

But these SLOs go both ways. 99.9% uptime, also means 0.1% downtime (about 43 minutes/month). Understanding what those limits are is necessary for both designing any HA/DR strategy and planning deployments/risk management. By using specific numbers, we can make practical, informed decisions.

“Downtime budgets” set realistic expectations with all parties, whether they be the developers of dependent systems, or the end users. Explicit SLOs encourage teams to plan maintenance and design deployment patterns that keep downtime levels within limits that are both acceptable and practical. These downtime budgets also allow teams to balance their appetite for innovation/risk against their stability responsibilities.

For example, if a team has used up most of their monthly downtime budget within the first week of the month, perhaps now is the time to focus their energy on tasks associated with improving stability, rather than pushing ahead with risky new features? Equally, if they’ve been routinely outperforming their SLOs by a couple of orders of magnitude, perhaps they should prioritise the next customer facing feature over further stability investments?

By publishing both the SLO and performance metrics for each service, teams become accountable for their own work. If specific SLOs are missed on a regular basis, this should trigger a conversation about how the business can support the team to improve. Perhaps the SLO is unreasonable? Perhaps the team needs a little support? In either case, it allows for the organisation to focus their energy and investments where they are most needed.   

## Bounded Contexts enable Bounded Autonomy

In an ideal world, where reviews are necessary, they should be carried out by someone who is able to read the code while it’s still fresh, understand the consequences, and provide insightful recommendations about improvements. Where regulatory compliance is concerned, the review should (as a minimum) be carried out by an engineer who understands the systems well enough to spot any potentially fraudulent changes. (Note: this generally requires a great deal of technical skill/experience.) Preventing fraud, after all, is the entire point of the Sarbanes-Oxley (SOX) Act.

Since Continuous Integration teaches us to prioritise merging over diverging (as discussed earlier in this series), the reviewer should also prioritise the code review over anything they are currently working on. Ideally, within minutes of the change being submitted for approval, the reviewer(s) should pause whatever they are doing and review the change.

That’s right. It should be *expected* that reviewers take a break from any new development work, in order to prioritise the merging and deployment of “dev-complete” work.

From this stance the idea of a weekly Change Advisory Board (CAB) meeting should sound as poisonous and inefficient as it is. The audacity of any CAB member to think that any WIP should sit around, festering for a week, to fit in with their own schedule, should be considered egregiously arrogant. What else are they doing with their time that is more important to the business than merging a week’s worth of putrid divergence? Delivering reliable updates is *literally the point of the job* – not a task to be put off until the end of the week. (I talked in more detail about why [Change Advisory Boards Don’t Work](https://octopus.com/blog/change-advisory-boards-dont-work) last year, so I won’t repeat myself further here.)

It’s clear that senior managers should not be reviewing deployments. They probably will not be able to drop whatever they are working on whenever someone submits a new pull request. Their role is to support healthy review practices, not to personally conduct the reviews.

Fortunately, by decoupling our systems, the number of stakeholders for any specific system should be significantly reduced. Why should an engineer from the sales system care about a release to the support system (and vice versa)? API dependencies should be codified in the test framework. If a dependency has been broken, that should be flagged automatically, and if a test is missing, that’s the responsibility of the team who are dependent on it.

The people who own the speed and stability objectives for any given service should live within the team itself. They should not need a CAB, or any other external approvers.

By bringing the reviews into the cross-functional team for a given service, we enable reviews to happen on a radically timelier basis. This does not necessarily mean they are being completed by the developer themselves, or even someone with the same role. The team is cross-functional, after all. Perhaps a senior developer or an infrastructure engineer conducts the review? Maybe for a database change it’s a database specialist or even a DBA?

This revised review process will be more likely to catch errors and/or fraudulent changes and it maintains any “separation of duty” requirements. If anything, it’s *more* compliant with any legislation that I’ve come across to date than traditional CAB-based approval practices, and it supports the delivery of high-quality IT services, rather than hinders it. It also supports collaboration and knowledge sharing within the team responsible for delivering and maintaining the service.

One common concern folks have about dropping the CAB, is a perceived loss of control, authority or oversight by senior management or security teams. This perception is unfounded. To those concerned folks, I encourage you to imagine a world where the status of each service, with respect to its SLO, is available on a live dashboard.

Any service that routinely misses their SLOs would be highlighted to the same folks who traditionally sat on the CAB. This would be a cue to start a conversation about why the SLOs are being missed and what could be done to help. This would allow all those senior management figures who care so deeply about minimising risk, to focus their attention and energy with the people and services where they can deliver the most value.

Finally, for any reader with an objection about scheduling releases and revealing updates to customers/users, I’m afraid I’m going to kick that can for another couple of posts. You’ll find answers to that objection in Part 7: Near-Zero Downtime Deploys.

## Firm cultural leadership, from the top, is not optional

At first, the adoption of publicly reported SLOs can make engineers feel vulnerable. This is especially true if they already work in politically toxic working cultures where failures lead to scapegoating rather than support. If everyone can see when I screw up, will I be fired as soon as I make a mistake? I mean, we all make mistakes. When I started this series, I thought it was a single blog post! Then it became a 3-parter… Now it’s grown into something *much* bigger and I’m a long way behind my original publication schedule!

The shift toward bounded responsibility can be hugely beneficial, but only if the team feel safe and supported, even in failure. This is something that can only be achieved from the top. Inspiring leaders, who value the psychological safety of their teams, are critical.

> *“No matter what the problem is, it’s always a people problem.”*
> *Gerald Weinberg*

## Next time

We started this series with a look at traditional database delivery hell, and the various factors that reinforce a vicious cycle that leads to a technical debt singularity. We then went on to imagine a better way, covering the theoretical foundations of resilience, continuous delivery, and loose coupling. However, while we went into some detail imagining what a better system might look like, I’ve not written in any technical detail about how to transition from one architecture to the other.

In my next post (post 6), we’ll switch gears. We’ll move from theory into practice, with the first of three technical posts intended to aid you to develop three important capabilities that will support your transition to a loosely coupled architecture.

In the first of these three posts we’ll talk about the replacement of shared dev/test instances with the self-service and on-demand provisioning of useable and regulatory compliant dev/test environments. After that, we’ll finish this series with two posts that discuss near-zero downtime releases and a safe process for breaking monoliths up into smaller services.

While each of these capabilities is individually valuable, it’s in the combination of all three that the benefits of each are multiplied.

!include <safe-schema-updates-posts>
