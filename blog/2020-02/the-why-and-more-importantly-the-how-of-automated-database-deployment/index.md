---
title: The why and more importantly the how of automated database deployment
description: Why and how your should automate your database deployments
visibility: public
published: 2020-02-12
metaImage: redgate-database-deployments.png
bannerImage: redgate-database-deployments.png
tags:
 - DevOps
 - Database Deployments
---

![Why and how your should automate your database deployments](redgate-logo.png)

Reliability, traceability, speed: these are the top three motivators for automating the deployment of database changes. Especially when it comes to Production, there is no compromising on the level of scrutiny that may just prevent something disastrous happening in Production.

This need for quality has a strange effect. We talked recently to 30 teams at various stages of automating database deployment, and we were surprised that for many teams the need for quality deployments prevented them automating at first.

After all, if something absolutely has to be right, you do it yourself. You tend to every step, you take your time, you use the full force of your experience, all the good things humans bring to a job, right? Moreover, Production changes are a highly visible part of the DBA role; it’s easy to think that automating would make a statement to the rest of the organization that it isn’t important.

Then, suddenly, something happens and those same people flip. Sometimes it’s a series of failed deployments, a recognition of human limitations, or maybe a new person joins the team who has worked in a better way. That same quality imperative now generates the opposite behavior, and the teams recognize a simple truth: **If you love something, automate it.**

This is not to say, ‘developers should go crazy with the database’. There’s an analogy with the old saw that if you love something, you should set it free; but the opposite is true here. (After all, free-range databases have a way of turning feral.) A high quality process constraining the flow of change eases further development, rather than inhibiting it. But where do you start?

In our research calls, process changes were rolled out in three different ways:

## Top-down

Some teams are (un?)lucky enough to have a top-down initiative to bring an army of wayward databases into line. In this case, a new system might be designed up-front, with all the components in place. This approach is favored where speed and efficiency can be tied directly to business value. Take one consumer-facing insurance website: new features and bug fixes were being delivered in an environment of intense competitive pressure. Speedy delivery into Production was essential, so they worked hard upfront to bring the database into their continuous delivery process.

If communication between developers and DBAs is poor, this may be the only approach that can get both sides working together to make the change.

## From the ‘left’

For teams who bring in change incrementally, automation solutions lend themselves nicely to being rolled out in stages. First, developers start tracking database changes in source control, solving issues around visibility and conflict. Once they’re basking in the warm light of traceability, the teams start running changes through their CI system, so that each database change triggers basic sanity checks. As with application development CI, this gives fast feedback to developers about their changes.

For each ‘green build’, some teams push changes straight to an integration or test environment (continuous deployment) or just make a package available (continuous delivery) for others to pick up.

Either approach, push or pull, can extend to downstream environments like UAT, Staging, Production, but in most situations, a release management tool such as Octopus Deploy formalizes the relationship between environments and gives visibility into what versions are where.

It might sound like this approach is always driven by developers, but we’ve seen a surprising number of DBAs instigate the change. Or maybe it’s not that surprising; repeated rehearsals are going to surface most problems in Test rather than Production, increasing overall reliability.

## Outside-in

Another pattern I’ve seen is to do automation by pincer movement, but I’ll be honest, I’m not sure this gives the best chance of a good outcome. In this case, automation comes in from both ends of the development lifecycle: Development and Production. The motivation for this is to ease the keenest pain points with tools or scripts, then let automation grow as everyone comes to trust the systems.

The reason I’m not so sure about this is that, from the teams I’ve talked to, it’s easy to get stuck at this point. Sure, they address the biggest problems first – generally a good approach. The downside is that the chaos is compressed into integration and test, areas which are notoriously difficult to budget the effort required to fix.

## Conclusion

For the teams we talked to, getting environments and process into shape was much more challenging than the automation itself, but even so, we couldn’t find anyone who would accept a step backwards away from deployment automation. Even moving jobs, they would insist on applying their automation knowledge to new situations. Putting the work into the deployment process, rather than individual deployments, has meant that all the love and care paid off hundreds of times over.

---

This is a guest post from Elizabeth Ayer from Redgate. Redgate is the leading provider of software for professionals working on the Microsoft data platform. They help over 800,000 people in every type of organization around the world, from small and medium sized businesses to 91% of companies in the Fortune 100. Their SQL Toolbelt helps users implement DevOps best practice for their databases, easily monitor database performance, and improve team productivity.

[Learn more](https://www.red-gate.com)