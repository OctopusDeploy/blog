---
title: "Change Advisory Boards Don’t Work"
description: Proper scrutiny is important, but CABs are an inefficient and ineffective way to scrutinize.
author: alex.yates@dlmconsultants.com
visibility: public
published: 2020-07-16
metaImage: change-advisory-boards-dont-work.png
bannerImage: change-advisory-boards-dont-work.png
bannerImageAlt: Change Advisory Boards Don’t Work
tags:
 - DevOps
---

![Change Advisory Boards Don’t Work](change-advisory-boards-dont-work.png)

In the name of quality, many organizations have a Change Advisory Board or Change Approval Board (CAB) who review changes before they are executed against production. They are sometimes set up to comply with some regulation, such as the 2002 Sarbanes-Oxley Act (SOX), and/or to enforce “separation of duties”. Other times CABs are introduced following a series of deployment failures in an effort to improve reliability.

The idea is to provide additional scrutiny, often in an effort to catch mistakes, poor code or fraudulent changes. It’s a noble goal but, unfortunately, CABs mostly do more harm than good.

This was articulately demonstrated in 2018 by Nicole Forsgren, Gene Kim, and Jez Humble in [Accelerate](https://www.amazon.co.uk/Accelerate-Software-Performing-Technology-Organizations/dp/1942788339). They analyzed the data from the 2014 – 2017 State of DevOps Reports and they:

> “found that external approvals were negatively correlated with lead time, deployment frequency, and restore time, and had no correlation with change fail rate. In short, approval by an external body (such as a manager or CAB) simply doesn’t work to increase the stability of production systems, measured by the time to restore service and change fail rate. However, it certainly slows things down. It is, in fact, worse than having no change approval process at all.”

The quotation above contains some loaded terms. Let’s discuss what they mean and why they are significant.

## Lead time

For many, the DevOps movement started with a vague awareness about the importance of getting changes into production quickly as well as reducing the amount of work in progress (WIP) at any one time. The name “DevOps” is a rejection of the typical anti-pattern of tossing changes over the “wall of confusion” between Dev and Ops, where changes would inevitably become tangled up in a political psychodrama that resulted in delays and mistakes.

Building on Lean manufacturing and Agile software development, folks recognized the enormous challenges and waste associated with long development cycles and large WIP levels. In the early days of the DevOps movement, it was common to hear folks affectionately refer to lead time as the time it took to get from “Aha! to Kerching!” They recognized that any businesses that could deliver on their ideas quickly, in small increments, would have a significant advantage in the marketplace. 

We no longer lived in a world where big beat small. Fast was beating slow. If you could deliver value early and often you had the power to disrupt the market. It was possible to start selling products much earlier in the development cycle, and gradually refine ideas based on real-world feedback. This significantly reduced the requirement for upfront capital investment and resulted in better products and services. The old mantra that a business could only pick two out of speed, quality, and cost was dead.

Short lead times go hand in hand with reducing WIP. If an organization tries to make either too many changes or too big a change in one go the change management overhead quickly becomes overbearing and the risk of failure skyrockets. It is much more efficient to have short development cycles with fewer and smaller changes than vice versa.

However, CABs often significantly increase both lead times and WIP since updates are delayed and batched up as they go through the formal approval process. For example, if the CAB meets once a week, a change that took only an hour to write may be delayed by 40 business hours, or 168 real hours, before it can be delivered to end users. The unintended consequence for the business is significantly poorer lead times and increased WIP which leads to poorer quality, slower feedback, higher development costs, and poorer cashflow.

## Deployment frequency

The frequency with which an organization deploys is not a hard concept to understand, but it is an important metric with some counter-intuitive consequences.

Every deployment carries some risk, and failed deployments are a horrible business. According to [The Visible Ops Handbook](https://www.amazon.co.uk/Visible-Ops-Handbook-Implementing-Practical/dp/0975568612/), [The DevOps Handbook](https://www.amazon.co.uk/Devops-Handbook-World-Class-Reliability-Organizations/dp/1942788002/), and various Gartner Reports, around 80% of production outages are the result of someone making a change. If an organization has suffered a series of expensive deployment failures it is natural to want to do them less often and to put in place additional measures to test and validate them.

CABs are often formed within this context. However, as Chuck Rossi observed while he was working at Facebook, as deployments grow beyond a certain size it’s almost impossible to execute them successfully. If lots of small changes are bundled together into a big deployment, that deployment becomes inherently more complicated, more risky, and harder to fix. He concludes that “if we want more changes, we need more deployments”.

Unfortunately, CABs often create a bottleneck where many changes are reviewed in a single batch and then deployed within a single deployment window, often by someone with little understanding about the context of each change. Hence, despite the very best of intentions, the reality is that CABs often increase the risk of painful deployment failures as a direct result of reducing the deployment frequency, leading to bigger, more complicated and riskier deployments.

## Restore time and Change Fail Rate

Restore time, or Mean Time to Restore (MTTR), is a measure for how quickly a team can recover from an outage. Since we expect that 80% of outages are the result of a change, it is wise to pay special attention to how quickly teams can recover from a failed deployment and to invest in work that improves MTTR.

However, historically organizations have paid more attention to Mean Time Between Failure (MTBF). While attitudes are changing, people are still more likely to talk about how often a deployment fails, rather than how quickly failures can be fixed. 

In [Database Reliability Engineering](https://www.amazon.co.uk/Databases-at-Scale-Operations-Engineering/dp/1491925949) Laine Campbell and Charity Majors refer to this as “Resilience vs Robustness”. They explain that when folks design robust systems to never/rarely break, the system becomes brittle. Since failures occur so rarely the team is ill-prepared when failures do happen and often the failures are complicated and difficult to understand and fix. In contrast, when teams accept that failures will occur and design a system to cope with failures, they are far more likely to recover quickly. They promote practices such as canary deployment patterns, automated fail-overs, chaos engineering, and enabling systems to automatically switch to a scaled down mode. For example, by gracefully turning off resource intensive features while a service is under heavy load. By embracing these sorts of resilient practices, deployment failures are likely to have significantly less impact.

Hence, while it is of course desirable for change failure rates to be low, attempting to remove all risk is a fool’s errand. You could invest your entire IT budget in avoiding failure, but failures would still happen. In the meantime, the overhead of seeking a perfect deployment record will likely cripple delivery times and innovation. As the old saying goes, as long as perfection is impossible or impractical, “perfection is the enemy of good”.

In contrast, the organizations that focus on MTTR first, and change failure rate second, will have a far healthier relationship with risk. If failed deployments become less of a problem and your team is able to fix them much more quickly, it becomes less important whether the change failure rate is 1% or 2%. It might as well be 25%, for the organization would probably still outperform an organization with poor MTTR in the eyes of its customers.

Unfortunately, most CABs place more importance on stopping the broken deployments than they do on creating a system where it is safer to fail. This is not effective.

## The four key metrics

Taken together these four concepts (lead time, deployment frequency, MTTR, and change failure rate) form Accelerate’s “four key metrics”. These metrics were selected because they are demonstrated to result in positive business outcomes.

Counter-intuitive as it sounds, those organizations that score highly with respect to the speed metrics (lead time and deployment frequency) are the same organizations that score highly with respect to the quality metrics (MTTR and change failure rate). And the organizations that perform better with respect to lead time, deployment frequency, and MTTR have better business outcomes as measured by profitability, productivity, and market share. (Interestingly, change failure rate seems less important than the other three metrics.)

Forsgren, Humble, and Kim have demonstrated that speed begets quality, quality begets speed and they both result in better business outcomes. If you have a CAB, the fact that CABs are negatively correlated against both the four key metrics and business outcomes should be a cause for concern to you, your IT managers, and your shareholders.

## An ideal code review process

Having code reviewed by a knowledgeable expert is undoubtedly an excellent way to simultaneously improve quality and avoid knowledge hoarding. It can also be a great way to promote mentoring, personal development, and collaboration with people who have different skills or experience (e.g. DBAs, InfoSec, etc). Code reviews are a wonderful thing and this blog post should not be interpreted as a criticism of this valuable practice.

The goals of most CABs are undoubtedly positive. It’s their implementation where the problems arise.

Code reviews work best when they are done by someone who understands the code and the context. For example, they could be done by another member of the development team. Perhaps the person who sits at the next desk or a more experienced colleague? If you are a web developer but you needed to update a stored procedure in the database, you might prefer to get a review from a DBA. It is far more effective for code to be reviewed by a knowledgeable expert who understands the context than by a committee of folks who do not.

Code reviews should also be conducted on small sets of changes. The more code someone is asked to review, the less carefully they’ll review it and the fewer suggestions they’ll make. Hence, reviewing large batches of changes, by committee, within a relatively short meeting, is unlikely to deliver high quality scrutiny or feedback. In contrast, frequent small code reviews are much more effective.

Whoever is going to review your work should understand the importance of short lead times and minimizing WIP. Simply put, reviews should be conducted quickly and finishing work is more important than starting work. Hence, if you are working on a new dev task, and someone asks you to review their “dev-complete” change, you should generally prioritize their code review over your own work. While this may introduce a context switching overhead, this is a lesser evil than the batching up of changes to be reviewed at the end of the week. Hence, the ideal reviewer should be subservient to the developer, rather than the other way around. Any reviewer should review changes in near real time, as soon as the changes are ready for review, rather than allowing the changes to build up for a bulk review on a cadence that suits the reviewer. If the reviewer is an individual that should be manageable, but if the reviewer is a large committee it becomes impractical.

## The CAB process

Unfortunately, CABs rarely reflect the ideal code review process described above. Typically, a broad range of deeply risk-averse characters from around the organization will gather in a conference room or over video conference and work through a long list of changes that have been batched up over the last week or so. The CAB may well include people who are not subject matter experts and who may not understand the context.

These meetings are likely to happen on a fixed schedule, so the work is made to fit around the people’s calendars, rather than the people prioritizing their time based on the tasks at hand.

In most CAB meetings individual changes will not get much scrutiny due to the number and size of the changes to be reviewed.

Each change might only be relevant to one or two members of the CAB so the rest of the attendees are likely to be wasting their time. 

CAB meetings tend to become a rubber-stamping exercise with the same one or two questions being asked about each item. “Will this cause downtime?” “Do you have a roll-back plan?” Good, next. 

Perhaps worst of all these meetings can become an exceedingly tedious and calendar hogging affair to all attendees.

Regardless of the initial intentions, CABs generally turn into a theater of security, rather than a genuine attempt to create more reliable systems. They tend to create significant disruption while offering limited value.

## Separation of duties, regulatory compliance, and fraud

Before we go on, please be clear about the fact that I am not a lawyer and that while I mentioned SOX earlier in this post, my advice about technical practices should not be interpreted as legal advice about any specific law. If you have any concerns about any legal matters you should seek professional legal advice. My advice is purely concerning the effectiveness of specific technical practices.

As mentioned at the top of this post, CABs often exist because of regulatory requirements, often due to a specific clause concerning “separation of duties”. These regulations generally exist to prevent fraud and they are important.

Unfortunately, CABs are normally an ineffective way to spot fraud and they put a terrible burden upon the organization for the reasons stated above. The ideal code review process described above is a far better defense against fraud as well as being a more effective way to deliver innovative software.

Without making any comments about any specific laws, in my experience, most laws concerning fraud prevention are less concerned about the format of the code review, and more concerned about its effectiveness and auditability. If you were to review the laws that govern you, you may well find that a traditional CAB is not strictly necessary and that a more lightweight but no less strictly audited review process, which more naturally fits within your development cycles and incorporates a more thorough review, may be more appropriate.  

For example, the ideal code review process described above is ideally suited to using pull requests in source control. This way approvers and their comments are audited, and often various rules can be configured to define the sorts of people who are required to review each change. It becomes impossible to update the main branch in source control without having conducted an appropriate code review.

Alternatively, you could audit your reviews in your deployment pipeline, such as by using “[manual intervention](https://octopus.com/docs/deployment-process/steps/manual-intervention-and-approvals)” steps in Octopus Deploy. These are fully audited and include details about who reviewed the change, when it was reviewed, and any comments are stored in the logs.

You might wish to use both source control pull requests and manual review steps in your deployment pipeline for different sorts of reviews, but be careful adding too many manual approval gates since this is likely to increase lead time, WIP, and will result in bigger and riskier deployments. Reviews are important, but it’s also important to keep them lightweight and to avoid unnecessary bureaucracy.

If you are under any doubt about your legal requirements, please seek professional legal advice.

## What to do if your organization is thinking about forming a CAB

First, remain calm. It’s likely that the people making these decisions have the best intentions. They simply don’t understand the anti-patterns that CABs are likely to create. You should try to educate them, without steamrolling them. Most people like to learn and to be persuaded, but folks rarely respond well to being beaten into submission.

You could start by teaching them about the counter intuitive nature of the four key metrics above. From here you could explain that you agree that appropriate oversight is important, and that you are keen to deliver the necessary oversight as effectively and efficiently as possible.

You might want to consider embracing the most risk-averse people and those who understand the major risks best. Get them on-side by asking them to help you to design automated ways to flag up potential issues within your automated test framework or deployment pipeline. The people who best understand the risks often make excellent allies.

Pay special attention to any form filling or ticketing systems associated with the CAB. These are often poorly designed and either don’t give the user the opportunity to provide all the necessary information or they introduce an intolerable overhead for even simple and low-risk changes. When this happens the process will inevitably break down and no-one wins.

Ensure that the CAB has a well-defined scope and that, if possible, low-risk, and possibly even medium-risk changes are granted a free pass to skip the CAB altogether. This will allow the CAB more time to focus on scrutinizing the most risky changes. For example, try to get a commitment that if a change fits the following criteria it can bypass the CAB:

- The dev team has kept within their “Error Budget” for the last 30 days (see [Site Reliability Engineering](https://www.amazon.co.uk/Site-Reliability-Engineering-Betsy-Beyer/dp/149192912X), [Chapter 3](https://landing.google.com/sre/sre-book/chapters/embracing-risk/) for further details about error budgets).
- All changes are deployed through an automated deployment pipeline.
- All changes have been peer reviewed by another member of the team.
- All changes have passed the relevant tests and the codebase maintains at least x% code coverage.

As well as saving the CAB time, this will incentivize the dev teams to invest in appropriate quality control and test automation in order to avoid going through the CAB.

Finally, it probably wouldn’t hurt to buy a copy of [Accelerate](https://www.amazon.co.uk/Accelerate-Software-Performing-Technology-Organizations/dp/1942788339) for each of the key decision makers. If they don’t want to read it, make a deal with them that you will read a book on their recommendation (and make sure you do!), if they promise to read a book that you recommend. Then agree a time to meet, perhaps over a fancy meal, and discuss both books. You might both learn something and it may prompt a friendly and valuable discussion.

## What to do if you already have a CAB

Having to dance to the beat of the CAB is hardly ideal. However, upon reading this blog post you should avoid jumping to any knee-jerk reactions. Without having invested in any lightweight approval processes, you may well have become reliant on the CAB for your high-risk changes.

Love it or hate it, any changes to your existing processes should be made iteratively and carefully. Developing your deployment process, just like building a piece of software, should be done in small batches to avoid accidentally making a disastrous sweeping change. 

It’s also important to acknowledge the human factors. You’ve probably got a bunch of valuable and skilled colleagues who are already members of or otherwise invested in the CAB – you want to bring them with you. There may also be some powerful individuals who are uncomfortable about the idea of scaling back or disbanding the CAB. It probably wouldn’t be a good idea to make enemies out of them.

As a first step, talk to the supporters of the CAB about how you can make it more efficient. After all, it’s highly likely that they are as bored of the CAB as you are. They can be pretty tedious meetings and most of the people involved probably see them as a distraction from more important tasks.

Perhaps you could ask them a few of the following questions:

- Are the changes triaged into high risk, medium risk, and low risk changes? What is the difference between a low-risk and a medium-risk change? Is the threshold set at an appropriate level that enables the CAB to focus their attention on the most important issues?
- What would it take for the CAB to be comfortable to auto-approve the low-risk changes or to allow them to bypass the CAB altogether?
- What could you do to downgrade a medium risk change to a low risk change? What sort of validation or testing would be required? Could these tests be automated in order to pre-approve the medium risk changes?
- How could any paperwork or forms be simplified and streamlined?
- Are all CAB members required to be present to review all changes? For example, if the CAB generally meets on a Wednesday afternoon, could you additionally run a scaled down version of the CAB daily at 11 a.m. with just one or two CAB members to review low/medium risk changes as soon as they are available? This would give you more time to scrutinize the high-risk changes in the main meeting on Wednesday and improve the lead time for low and medium risk changes. This might significantly reduce WIP and drive improvements against all four key metrics.
- What architectural changes could you invest in to decouple systems from each other? This could reduce the risk of one failure affecting multiple systems. Hence, fewer people might be required to review each change.
- What investments could you make to improve MTTR or otherwise de-risk your deployments? If the consequences of failed deployments were reduced significantly, would that enable the CAB to allow changes to flow to production more freely?
- How could you report on the success/failure of any changes that you make to the CAB process? Can you track the four key metrics, or any other key performance metrics, as you iterate on your CAB processes to ensure that your changes are having a positive effect?

---

Alex Yates has been helping organisations to apply DevOps principles to their data since 2010. He’s most proud of helping Skyscanner develop the ability to  [deploy 95 times a day](https://www.youtube.com/watch?v=sNsPnCv7hHo) and for supporting the United Nations Office for Project Services with their release processes. Alex has worked with clients on every continent except Antarctica – so he’s keen to meet anyone who researches penguins.
 
A keen community member, he co-organises [Data Relay](https://datarelay.co.uk/), is the founder of [www.SpeakingMentors.com](http://www.speakingmentors.com/) and has been recognised as a  [Microsoft Data Platform MVP](https://mvp.microsoft.com/en-us/PublicProfile/5002655?fullName=Alex%20Yates) since 2017.
 
Alex is the founder of [DLM Consultants](http://dlmconsultants.com/), an official Octopus Deploy partner. He enjoys mentoring, coaching, training and consulting with customers who want to achieve better business outcomes through improved IT and database delivery practices.
 
If you would like to work with Alex, email: [enquiries@dlmconsultants.com](mailto:enquiries@dlmconsultants.com) 