---
title: "Change Advisory Boards Don't Work"
description: Proper scrutiny is important, but CABs are an inefficient and ineffective way to scrutinize.
author: alex.yates@dlmconsultants.com
visibility: public
published: 2020-07-10
metaImage: 
bannerImage: 
tags:
 - DevOps
---

In the name of quality, many organisations have a Change Advisory Board or Change Approval Board (CAB) who review changes before they are executed against production. They are sometimes set up in response to a legal requirement to enforce “separation of duties”. Often this is in response to the 2002 Sarbanes-Oxley Act (SOX) or some other regulation. Other times CABs are introduced following a series of deployment failures in an effort to improve reliability.

The idea is to provide additional scrutiny in order to catch mistakes, poor code and/or fraudulent changes. It’s a noble goal but, unfortunately, CABs mostly do more harm than good.

This was articulately demonstrated in 2018 by Nicole Forsgren, Gene Kim and Jez Humble in [Accelerate](https://www.amazon.co.uk/Accelerate-Software-Performing-Technology-Organizations/dp/1942788339). They analysed the data from the 2014 – 2017 State of DevOps Reports and they:

> “found that external approvals were negatively correlated with lead time, deployment frequency, and restore time, and had no correlation with change fail rate. In short, approval by an external body (such as a manager or CAB) simply doesn’t work to increase the stability of production systems, measured by the time to restore service and change fail rate. However, it certainly slows things down. It is, in fact, worse than having no change approval process at all.”

The quotation above contains some loaded terms. Let’s discuss what they mean and why they are significant.

## Lead time

For many, the DevOps movement started with a vague awareness about the importance of getting changes into production quickly. The name “DevOps” is a rejection of the typical anti-pattern where changes would be tossed over the “wall of confusion” between Dev and Ops. Changes would inevitably become tangled up in a political psychodrama that resulted in delays and mistakes.

Building on Lean manufacturing and Agile software development, folks recognised the enormous challenges and waste associated with long development cycles. In the early days of the DevOps movement it was common to hear folks affectionately refer to lead time as the time it took to get from “Aha! to Kerching!” They recognised that any businesses that could deliver on their ideas quickly, in small increments, would have a significant advantage in the marketplace. 

We no longer lived in a world where big beat small. Fast was beating slow. If you could deliver value early and often you had the power disrupt the market. It was possible to start selling products much earlier in the development cycle, and gradually refine ideas based on real-world feedback. This significantly reduced the requirement for up front capital investment and resulted in better products and services. The old mantra that a business could only pick two out of speed, quality and cost was dead.

Short lead times go hand in hand with reducing Work in Progress (WIP). If an organisation tries to make either too many changes or too big a change in one go the change management overhead quickly becomes overbearing and the risk of failure skyrockets. It is much more efficient to have short development cycles with fewer and smaller changes than vice versa.

However, CABs often significantly increase both lead times and WIP as updates are delayed and batched up as they go through the formal approval process. If the CAB only meets once a week, a change that took only an hour to write may be delayed by 40 business hours, or 168 real hours, before it can be delivered to end users. The unintended consequence for the business is significantly poorer lead times and increased WIP which leads to poorer quality, slower feedback and higher development costs.

## Deployment frequency

The frequency with which an organisation deploys is not a hard concept to understand, but it is an important metric with some counter-intuitive consequences.

Every deployment carries some risk, and failed deployments are a horrible business. According to [The Visible Ops Handbook](https://www.amazon.co.uk/Visible-Ops-Handbook-Implementing-Practical/dp/0975568612/), [The DevOps Handbook](https://www.amazon.co.uk/Devops-Handbook-World-Class-Reliability-Organizations/dp/1942788002/) and various Gartner Reports, around 80% of production outages are the result of someone making a change. If an organisation has suffered a series of expensive deployment failures it is natural to want to do them less often and to put in place additional measures to test and validate them.

CABs are often formed within this context. However, as Chuck Rossi observed while he was working at Facebook, as deployments grow beyond a certain size it’s almost impossible to execute them successfully. If lots of small changes are bundled together into a big deployment, that deployment becomes inherently more complicated, more risky and harder to fix. He concludes that “if we want more changes, we need more deployments”.

All deployments carry risk, but smaller and more frequent deployments are less risky and, when they do fail, they are much easier to fix.

Unfortunately, CABs often create a bottleneck where many changes are reviewed in a single batch and then deployed within a single deployment window by someone with little understanding about the context of each change.

It is easy to understand how, with best intentions, a push for more reliable deployments may lead to the formation of a CAB. However, this is likely to lead to a slower release cadence with bigger and more complicated deployments which are more likely to fail. 

## Restore time and Change Fail Rate

Restore time, or Mean Time to Restore (MTTR), is a measure for how quickly a team can recover from an outage. Since we expect that 80% of outages are the result of a change, it is wise to pay special attention to how quickly teams can recover from a failed deployment and to invest in work that improves MTTR.

However, historically organisations have paid more attention to Mean Time Between Failure (MTBF). While attitudes are changing, people are still more likely to talk about how often a deployment fails, rather than how quickly failures can be fixed. 

In [Database Reliability Engineering](https://www.amazon.co.uk/Databases-at-Scale-Operations-Engineering/dp/1491925949) Laine Campbell and Charity Majors refer to this a “Resilience vs Robustness”. They explain that when folks design robust systems to never/rarely break, the system becomes brittle. Since failures occur so rarely the team is ill-prepared when they do happen and often the failures are complicated and difficult to understand and fix. In contrast, when teams accept that failures will occur and design a system to cope with failures, they are far more likely to be able to recover quickly. They promote practices such as canary deployment patterns, automated fail-overs, chaos engineering and enabling systems to automatically switch to a scaled down mode. (For example, by gracefully turning off resource intensive features while a service is under heavy load.) By embracing these sorts of resilient practices deployment failures are likely to have significantly less impact.

Hence, while it is of course desirable for change failure rates to be low, attempting to remove all risk is a fool’s errand. You could invest your entire IT budget into avoiding failure, but failures would still happen. In the meantime, the overhead of seeking a perfect deployment record will likely cripple delivery times and innovation. As the old saying goes, as long as perfection is impossible or impractical, “perfection is the enemy of good”.

In contrast, the organisations that focus on MTTR first, and change failure rate second, will have a far healthier relationship with risk. If failed deployments become less of a problem and your team is able to fix them much more quickly, it becomes less important whether the change failure rate is 1% or 2%. It might as well be 25%, for the organisation would probably still outperform an organisation with poor MTTR in the eyes of its customers.

Unfortunately, most CABs place more importance on stopping the broken deployments than they do on creating a system where it is safer to fail. This is not effective.

## The Four Key Metrics

Taken together these four concepts (lead time, deployment frequency, MTTR and change failure rate) form Accelerate’s “four key metrics”. These metrics were selected because they are demonstrated to result in positive business outcomes.

Counter-intuitive as it sounds, those organisations that score highly with respect to the speed metrics (lead time and deployment frequency) are the same organisations that score highly with respect to the quality metrics (MTTR and change failure rate). And the organisations that perform better with respect to lead time, deployment frequency and restore time have better business outcomes as measured by profitability, productivity, and market share. (Interestingly, change failure rate seems less important.)

Forsgren, Humble and Kim have demonstrated that speed begets quality, quality begets speed and they both result in better business outcomes. If you have a CAB, the fact that CABs are negatively correlated against the four key metrics and positive business outcomes should be a cause for concern to you, your IT managers, and your shareholders.

## An ideal code review process

Having code reviewed by a knowledgeable expert is undoubtedly an excellent way to simultaneously improve quality and avoid knowledge hoarding. It can also be a great way to promote mentoring, personal development and/or collaboration with people who have different skills or experience (e.g. DBAs, InfoSec etc). Code reviews are a wonderful thing and this blog post should not be interpreted as a criticism of this valuable practice. The goals of most CABs are undoubtedly positive. Their implementation is the problem.

Code reviews work best when they are done in a timely manner and by someone who understands the code and the context. For example, they could be done by another member within the development team, perhaps the person who sits at the next desk. If you are a web developer but you needed to update a stored procedure in the database, you might prefer to get a review from the DBA. It is far more effective for code to be reviewed by a knowledgeable expert who understands the context than by a committee of folks who do not.

Whoever is going to review your work should understand the importance of short lead times and minimising WIP. Simply put, finishing work is more important than starting work. If you are working on one change, and someone asks you to review their change, you should generally prioritise their code review over your own work. While this may introduce a context switching overhead, this is a lesser evil than the batching up of code reviews to be approved at the end of the week. 

Code reviews should also be conducted on small sets of changes. The more code someone is asked to review, the less carefully they’ll review it and the fewer problems they’ll spot.

## The CAB process

Unfortunately, CABs rarely promote the ideal code review process described above. Typically, a broad range of deeply risk-averse characters from around the organisation will gather in a conference room or over video conference and work through a long list of changes in batches. The CAB may well include people who are not subject matter experts and who may not understand the context.

These meetings are likely to happen on a fixed schedule, so the work is made to fit around the people’s calendars, rather than the people prioritising their time based on the tasks at hand.

In most CAB meetings individual changes will not get much scrutiny due to the number and size of the changes to be reviewed.

Each change might only be relevant to one or two members of the CAB so the rest of the attendees are likely to be wasting their time. 

CAB meetings tend to become a rubber-stamping exercise with the same one or two questions being asked about each item. “Will this cause downtime?” “Do you have a roll-back plan?” Good, next. 

Perhaps worst of all these meetings can become an exceedingly tedious and calendar hogging affair to all attendees.

Regardless of the initial intentions, CABs generally turn into a theatre of security, rather than a genuine attempt to create more reliable systems. They tend to create significant disruption while offer limited value.

## Separation of duties, regulatory compliance and fraud

Before we go on, please be clear about the fact that I am not a lawyer and that while I may have mentioned SOX earlier in this post, my advice about technical practices should not be interpreted as legal advice about any specific law. If you have any concerns about any legal matters you should seek professional legal advice. My advice is purely concerning the effectiveness of specific technical practices.

As mentioned at the top of this post, CABs often exist because of regulatory requirements, often due to a specific clause concerning “separation of duties”. These regulations generally exist to prevent fraud and they are important.

Unfortunately, CABs are normally an ineffective way to spot fraud and they put a terrible burden upon the organisation for the reasons stated above. The ideal workflow described above is a far better defence as well as being a more effective way to deliver innovative software.

Without making any comments about any specific laws, in my experience most laws concerning fraud prevention are less concerned about the format of the code review, and more concerned about the fact that changes have been reviewed by an appropriate person and that the review has been appropriately audited. If you were to review the laws that govern you, you may well find that a traditional CAB is not strictly necessary and that a more lightweight but no less strictly audited review process, which more naturally fits within your development cycles and incorporates a more thorough review, may be more appropriate.  

For example, the ideal code review process described above is ideally suited to using pull requests in source control. This way approvers and their comments are audited, and often various rules can be configured to define the sorts of people who are required to review each change. It becomes impossible to update the main branch in source control without having conducted a code review.

Alternatively you could audit your reviews in your deployment pipeline, such as by using “[Manual Intervention](https://octopus.com/docs/deployment-process/steps/manual-intervention-and-approvals)” steps in Octopus Deploy. These are fully audited and details about who reviewed the change, when it was reviewed, and any comments are stored in the logs.

You might wish to use both source control pull requests and manual review steps in your deployment pipeline for different sorts of reviews, but be careful adding too many manual approval gates since this is likely to increase lead time, WIP and will result in bigger and riskier deployments.

If you are under any doubt about your legal requirements, my advice is to seek professional legal advice and then to design a review process that meets all your legal requirements and which looks as much like the ideal code review process as is legally appropriate.

## What to do if your organisation is thinking about forming a CAB

First, remain calm. It's likely that the people making these decisions have the best intentions. They simply don't understand the anti-patterns that CABs are likley to create. You should try to educate them, without steamrolling them. Most people like to learn and to be persuaded, but folks rarely respond well to being beaten into submission.

You could start by teaching them about the counter intuitive nature of the four key metrics above. From here you could explain that you agree that appropriate oversight is important, and that you are simply looking for the most effective and efficient way to deliver that oversight.

You might want to consider embracing the most risk-averse people and those who understand the major risks best. Get them on-side by asking them to help you to design automated ways to flag up potential issues within your automated test framework or deployment pipeline. The people who understand the risks best make excellent allies.

Pay special attention to any form filling or ticketing systems associated with the CAB. These are often poorly designed and either don't give the user the opportunity to provide all the necessary information or they introduce an intolerable overhead for even simple and low-risk changes. When this happens the process will inevitably break down and no-one wins.

Ensure that the CAB has a well-defined scope and that, if possible, low-risk, and possibly even medium-risk changes are granted a free pass to skip the CAB process. This would allow the CAB more time to focus on scrutinising the most risky changes. For example, try to get a commitment that if a change fits the following criteria it can bypass the CAB:

- The dev team has kept within their “Error Budget” for the last 30 days (see [Site Reliability Engineering](https://www.amazon.co.uk/Site-Reliability-Engineering-Betsy-Beyer/dp/149192912X), [Chapter 3](https://landing.google.com/sre/sre-book/chapters/embracing-risk/) for further details about error budgets)
- All changes are deployed through an automated deployment pipeline
- All changes have been peer reviewed by another member of the team
- All changes have passed the relevant tests and the codebase maintains at least x% code coverage  

As well as saving the CAB time, this will incentivise the dev teams to invest in appropriate quality control and test automation in order to avoid going through the CAB.

Finally, it probably wouldn't hurt to buy a copy of [Accelerate](https://www.amazon.co.uk/Accelerate-Software-Performing-Technology-Organizations/dp/1942788339) for each of the key decision makers. If they don't want to read it, make a deal with them that you will read a book on their recommendation (and make sure you do!), if they promise to read a book that you recommend. Then agree a time to meet, perhaps over a fancy meal, and discuss both books. You might both learn something and it may prompt a friendly and valuable discussion.

## What to do if you already have a CAB

Having to dance to the beat of the CAB is hardly ideal. However, upon reading this blog post you should avoid jumping to any knee-jerk reactions. Without having invested in any lightweight approval processes, you may well have become reliant on the CAB for your high-risk changes.

Love it or hate it, any changes to your existing processes should be made iteratively and carefully. Developing your deployment process, just like building a piece of software, should be done in small batches to avoid accidentally making a disastrous sweeping change. 

It’s also important to acknowledge human factors. You’ve probably got a bunch of valuable and skilled colleagues who are already members of or otherwise invested in the CAB – you want to bring them with you. There may also be some powerful individuals who are uncomfortable about the idea of disbanding the CAB. It probably wouldn’t be a good idea to make enemies out of them.

As a first step, talk to the supporters of the CAB about how you can make it more efficient. After all, it’s highly likely that they are as bored of the CAB as you are. They can be pretty tedious meetings and most of the people involved probably see them as a distraction from more important tasks.

Perhaps you could ask them a few of the following questions:

- Are the changes triaged into high risk, medium risk and low risk changes? What is the difference between a low-risk and a medium-risk change? Is the threshold set at an appropriate level that enables the CAB to focus their attention on the most important issues?
- What would it take for the CAB to be comfortable to auto-approve the low-risk changes or to allow them to bypass the CAB altogether?
- What could you do to downgrade a medium risk change to a low risk change? What sort of validation or testing would be required? Could these tests be automated in order to pre-approve the medium risk changes?
- How could any paperwork or forms be simplified and streamlined?
- Are all CAB members required to be present to review all changes? For example, rather than all meeting on a Wednesday, could you run a scaled down version of the CAB with just one or two CAB members for low/medium risk changes each morning at 11am? This would give you more time to scrutinise the high-risk changes in the main meeting on a Wednesday and improve the lead time for low and medium risk changes.
- What architectural changes could you make to decouple systems from each other such that the risk of one failure affecting other systems and the necessity to seek the approval of so many people is reduced.
- What investments could you make to improve MTTR or otherise de-risk your deployments? If the consequences of failed deployments were reduced significantly, would that enable the CAB to allow low/medium risk changes to flow to production more freely?
- How could you report on the success/failure of any changes that you make to the CAB process? Are you able to track the four key metrics as you iterate on your CAB processes to ensure that your changes are having a positive affect?