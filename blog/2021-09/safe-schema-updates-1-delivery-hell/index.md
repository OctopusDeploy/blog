---
title: Safe schema updates - Database delivery hell
description: This post opens a series about safe schema updates with a brief tour through Dante's 9 levels of Database Delivery Hell.
author: alex.yates@dlmconsultants.com
visibility: public
published: 2021-09-07-1400
metaImage: blogimage-databasedeliveryhell-2021.png
bannerImage: blogimage-databasedeliveryhell-2021.png
bannerImageAlt: nine level database server with fire on the top level
isFeatured: false
tags:
 - DevOps
 - Database Deployments
 - Deployment Patterns
---

This blog post is part 1 of a series about safe schema updates. Links to the other posts in this series are available below:

!include <safe-schema-updates-posts>

In order to understand why it’s necessary to make a change, it’s useful to reflect on where we are right now. Apologies in advance, this might make for uncomfortable reading.

*“That’s going to require a database change.”*

IT folk often shudder at those words. Years of experience have taught them to expect an uphill battle to get *that* feature shipped to production.

There are several reasons for this sense of impending doom. For fun, I’m going to borrow a little artistic license from Dante. (For the record, I totally stole this idea from [Gianluca Sartori](https://spaghettidba.com/) - he's on [Twitter](https://twitter.com/spaghettidba), and you can check out [his devilishly stylish database “infernals” talk](https://www.youtube.com/watch?v=p1qQlmoj0sE).)

**Level 0: Data hell**

The unique challenge with databases is the data. Business-critical information is not saved in source control, so it’s impossible to delete and redeploy the database in the same way we might remove a troublesome web server.

Hence, database rollbacks aren’t easy. Arguably, there’s no such thing as a database rollback. If a production deployment goes bad, it might be necessary to restore a backup. That’s going to result in downtime and (possibly) data loss. That could be a disaster for your users and expensive for the business.

But this isn’t just a story about production. The dev/test databases rarely have truly representative data. Hence, bugs and performance issues are often only discovered in production. What’s more, if it’s impossible to reliably dry-run deployments in a “production-like” environment, it’s difficult to test for data loss or poorly performing deployment scripts.

When we fail to make representative and safe test data available in dev/test environments and when we fail to test for data issues in deployments, we disrespect our data. This sin has consequences.

**Level 1: Dependency hell**

Far too often “the database” becomes a shared back-end service for a myriad of dependent systems. Battle-worn engineers have learned the hard way that changing the schema can have unintended consequences on those dependent systems. It’s impossible for an engineer to be confident about a change when they don’t even know what their dependent systems are. Unfortunately, these dependencies are rarely well documented or tested.

This is especially bad when dependencies start to crop up between databases, for example, through Stored Procedures, Views or (shudder) Linked Servers. The worst offenders might even see dependencies between their dev, test and production environments. 

Production deployment issues are exacerbated by the fact that, thanks to all the dependencies, dev/test environments are very hard to provision and maintain. 

In an ideal world, dev/test environments would mostly be disposable, dedicated environments that are spun up and discarded by developers for each new dev task. However, given the time and effort that is usually required to build a large or complicated database, dev/test servers are often shared by large groups, making change control difficult and rendering it impossible to test changes in isolation.

These shared dev/test “wild-west” dumpster fires quickly become inconsistent with the production systems. Hence, they cannot be trusted as realistic representations of production, and any dev/test work conducted on them is fundamentally unreliable. 

**Level 2: Global failure hell**

The schema deployment itself is especially risky because, thanks to the dependencies, the database has become a single point of failure for so many services. A single forgotten WHERE clause or performance-draining CURSOR could have global consequences.

As mentioned above, due to the complexity of dependencies, there’s rarely a fit-for-purpose testing environment. What’s more, having a reliable automated test suite for each dependent service is unlikely and (probably) unfeasible. This makes it impossible to have confidence that nothing is going to break when the deployment is executed.

:::hint
These problems are real and significant. These first three levels are tied together nicely in one of the massive deployment failures in [The Phoenix Project](https://octopus.com/blog/devops-reading-list#phoenix). There was a missing index on a giant table in a single-point-of-failure database, at the centre of a tangled web of dependencies. This was probably missed either because the dev/test databases didn’t match production, or because they did not have representative data, so the performance issues weren’t spotted.

The update was running agonizingly slow, and it couldn’t be cancelled. They missed their downtime window and, due to the enormous number of dependent services, they caused enormous disruption to thousands of internal users and customers when the systems did not come back online on Monday morning.

I’ll back up that fictional disaster with a very real one. I once worked for a company that had one of these wild-west, single-point-of-failure, shared databases underpinning their dev environment. It was critical for the 100+ developers to test the stuff they were working on with realistic data. One time, someone accidentally dropped all the SQL logins. The entire dev function, as well as the dependent services, were locked out. It took the DBAs over a week to fix it, because there was a show-stopping issue in production at the same time.

All those developers were twiddling their thumbs for a week. I’m nervous to imagine what the typical annual budget of a 100+ dev team in the finance sector looks like, but I imagine a week of thumb twiddling didn’t go down well with the shareholders. 
:::

**Level 3: Release coordination hell**

It would be bad enough if you were just deploying the database change. However, due to the dependencies, you may also need to deploy new versions of a collection of dependent systems at the same time, or in a specific order. The entire process needs to be orchestrated carefully and an issue with just one part could jeopardize the whole exercise. 

This issue is exacerbated by poor source control practices and the use of shared environments. In such cases, the changes to be released may need to be cherry-picked from a larger set of changes that exist in a dev/test database. This, often manual, process requires a great deal more complexity and has more potential for error. It probably also involves either a horribly convoluted or an unrealistically simple branching pattern in source control. Either case creates complexity, risk and migraines. [I ranted about this topic in more detail on my personal blog](http://workingwithdevs.com/branching-reality/).

When releases of dependent objects/systems need to be carefully orchestrated, it’s a sign that you are suffering from the combination of a dependency nightmare in your code, as well as your team/project management structures. [Team Topologies](https://octopus.com/blog/devops-reading-list#tt) talks about these issues in more detail.

**Level 4: Downtime window hell**

Due to all the dependencies and the coordination effort above, you need to take the entire system offline for a period to complete the update. Negotiating downtime with users/customers/[“The Business”](https://twitter.com/paulstovell/status/1323552178091433984) is not easy, so you are forced to do it during unsociable hours. You might not be fully awake, and support is less likely to be available. (The developers who wrote the code might be asleep!) If you miss your deadline, there will be consequences.

To make matters worse, since you are only offered a limited number of downtime windows, you are under additional pressure to deliver as many changes as possible during each window. Deployments get larger, more complicated, and more risky.

:::warning
A variation of this hell is caused by naïve business assumptions about 100% uptime. 100% uptime is both practically impossible and unimaginably expensive. [Oftentimes senior management do not realize this](https://www.brentozar.com/archive/2011/12/letters-that-get-dbas-fired/). Through poor communication, tech folks are often left in a hopeless position, being measured against absurd expectations. This causes its own frustrating politicking and bad decisions.
:::

**Level 5: Bureaucracy hell**

Given the number of people who are potentially affected by any change to a monolithic back-end database, coupled with the severe consequences of failure, a lot of stakeholders want a veto on the deployment. Engineers are forced to spend as much time demonstrating that they have done the testing as they spend actually doing the testing. 

While this abundant caution might sound wise, it is usually ineffective when implemented badly. ([And it’s almost always implemented badly](https://octopus.com/blog/change-advisory-boards-dont-work).) Senior leadership is unlikely to accept a slower pace of work. Thus, if the testing/approval measures for changes cause delays, the consequence will be a significantly greater amount of “work in progress” (WIP). This leads to larger, and even more complicated deployments, with more dependency and coordination issues.

According to [Accelerate](https://octopus.com/blog/devops-reading-list#accelerate), data from the State of DevOps reports demonstrates that Change Advisory Boards are on average “worse than having no change approval process at all”. Despite good intentions, these safety measures result in demonic deployments that are more complicated and riskier.

**Level 6: Disobedience of bureaucracy hell**

Under pressure to release on time, and through personal desire to complete a job, engineers attempt to circumvent official processes. Middle managers play politics to protect their teams’ changes. People massage the truth to avoid bureaucracy. Shadow IT emerges because using the officially sanctioned systems is frustratingly tedious, hampering a team’s ability to hit their deadlines.

Senior managers may even support and congratulate such “innovation”, without acknowledging that they are contributing to a [big ball of mud](https://en.wikipedia.org/wiki/Big_ball_of_mud). Such short-term progress normally comes at the expense of long-term performance. 

**Level 7: Negligence hell**

As the business becomes more desperate to hit ever more impossible deadlines on increasingly stretched budgets, it becomes harder to invest in anything that isn’t directly related to narrow and specific objectives. At first this might not be so bad. It focuses energy on the most important work. However, inevitably this leads to under-investment in critical infrastructure. 

To use the “four types of work” terminology from [The Phoenix Project](https://octopus.com/blog/devops-reading-list#phoenix): This is the point that IT folks are under so much pressure to complete “Business Projects” and “Unplanned Work” that they have no time left to focus on “Internal Projects” or even routine “Changes” (like patching servers).

As [my fellow brits might put it](https://twitter.com/DavidJPoole/status/1361340954700107786), senior managers become “penny-wise, pound-foolish”. Improvement work and even routine maintenance is delayed or scrapped. Firefighting becomes commonplace. The team has stopped actively working on improving. They don’t have time.

One common engineering consequence is the short-sighted cheat to avoid refactoring or deleting anything in the database. Database deployments are certainly easier if you never clean up after yourself.

The idea that you shouldn’t delete stuff in the database is widespread in tech, and it needs to be challenged. This hack considers the most important part of your IT infrastructure to be a dumping ground for all your obsolete or misguided design choices from years past. This might save you a bit of time in the short term, but it’s going to bite. Refactoring is an essential part of software development. 

:::hint
I once worked for a company where all the tables had unintelligible four-character names. I asked why this was. Apparently, some previous database technology from decades-past had this limitation. Their current RDBMS was not restricted in this way, but they were so afraid of making changes that some developers still persisted with the four-character convention - even for new tables. It sure looks pretty if all your tables line up neatly in your IDE. 

This database is crucial for many important services, but using and maintaining it is almost impossible, especially for new recruits. No-one knows what the “QACD” or the “FFFG” tables are for. And those 5-table joins are impossible to decipher.
:::

Without refactoring our databases as our software and business needs evolve, we are sure to produce monolithic, big-ball-of-mud databases which are difficult to work with and full of (supposedly) deprecated code and unnecessary dependencies.

This is a one-way ticket to…

**Level 8: The technical debt singularity**

“Unplanned Work” has devoured everything.

As the business spirals down the levels, acquiring ever more [technical debt](https://martinfowler.com/bliki/TechnicalDebt.html), the percentage of each engineer’s time that is spent firefighting increases. Eventually that percentage hits 100%, or even higher, with engineers working overtime just to maintain core operations. A short-sighted solution might be to hire more people, but that won’t work. Frederick Brooks explained why in [The Mythical Man-Month](https://en.wikipedia.org/wiki/The_Mythical_Man-Month) in 1975! After nearly half a century, I’m not going to waste any more words repeating his ideas in this post. If you work in IT and you haven’t heard of it, I recommend that you follow the link above.

Unfortunately, the problems still get worse. There’s barely any time to work on new stuff.  We’ve reached the bottom of the pit. Some sort of change is inevitable. Either the business will change its thinking, or it’s a matter of time before they lose out to a more capable competitor.

In my experience, each level pushes folks toward the next. They reinforce each other. For those in the trenches, these issues can feel like [Sisyphus’s boulder](https://www.ancient.eu/sisyphus/), except this boulder is getting bigger and heavier over time. Reaching the summit, or even maintaining the current momentum, feels more impossible every day.

Something has to give. 

Wherever you are on your journey, it’s critical to recognize your trajectory and take a different path where necessary. The longer you wait, the more difficult it will be to escape. And, if those with the power to enact change don’t modify the way they think, escape will surely be impossible.

## Next time

The next post (part 2) will be the first of four posts intended to help folks to re-evaluate the way they view and assess safety within complex IT systems. We’ll start to imagine what a safer software architecture, delivery process, and development culture might look like. We'll begin by exploring the nature of failure within complex systems, before moving on to discuss the concepts of resilience, robustness and loose-coupling. 

Links to the other posts in this series are available below:

!include <safe-schema-updates-posts>
 
## Watch the webinars 

Our first webinar discussed how loosely coupled architectures lead to maintainability, innovation, and safety. Part two discussed how to transition a mature system from one architecture to another. 

### Database DevOps: Imagining better systems

<iframe width="560" height="315" src="https://www.youtube.com/embed/oJAbUMZ6bQY" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### Database DevOps: Building better systems

<iframe width="560" height="315" src="https://www.youtube.com/embed/joogIAcqMYo" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Happy deployments!
