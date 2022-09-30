---
title: DevOps, SRE, and platform engineering
description: Why DevOps, site reliability engineering, and platform engineering appear to be in conflict with each other, but work in harmony.
author: steve.fenton@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - tag
---

When it first arrived, DevOps was vague and poorly defined. More recently, attempts have been made to define DevOps based on studying the differences between high-performing organizations and their lower-performing peers.

We now have a clear picture of *good DevOps*, but also several other approaches that seem to compete or conflict with DevOps, like site reliability engineering (SRE) and platform engineering.

In this article you'll find out the differences between these approaches and what problems they intend to solve. You'll then see how they fit together as part of a high performance software delivery effort that is designed around your specific situation.

## Variations on a DevOps theme

Because DevOps didn't have a very specific definition on day one, lots of different set-ups have been given given the DevOps label. You'll find some organizations that look the same as they did before DevOps, some that have new roles, and others that have drastically changed their structure. How can all of these vastly different approaches all be called DevOps?

The answer to this question is that there are many ways to organize people around the goal of delivering software quickly and safely into the hands of users. The objective for early DevOps was to remove the conflicting goals of development and operations, so any organization that has aligned these two disciplines and increased collaboration between the people writing, deploying, and operating software is entitled to claim the DevOps label.

There is no specific requirement to change team structures or introduce new roles, so you might find any of the following set ups:

1. A development team and an operations team working together to deliver software
1. A cross-functional team who are responsible for everything, or "you build it, you run it"
1. A development team, an operations team, and a third "DevOps" team working in the middle

Each of these can be successful or represent an anti-pattern, but you can't determine this from outside of the organization because it very much depends on context that you can't see and whether their chosen set-up results in an improvement to software delivery and organizational performance.

Organizations are also on a journey, so what they do today may be an improvement on what they were doing last year even though it doesn't meet your definition of a DevOps organization. They might look more like  you expect in the future, depending on whether they can resolve issues with their culture, architecture, and available skills.

So, you should judge any specific organization by the trajectory of their improvement process, rather than on their current state.

## Patterns and anti-patterns

Christopher Alexander developed patterns for building and architecture during the 1970s, and his work is often referenced by the software industry when they refer to design patterns. The first set of reusable software design patterns was created by the gang of four (Erich Gamma, Richard, Helm, Ralph Johnson, and John Vlissides) in 1994 and many remain in use today.

Inspired by these design patterns, Andrew Koenig created a set of anti-patterns in 1995. These documented commonly used processes and actions that appeared to solve problems, but instead created more.

When it comes to DevOps, people are quick to label a particular structure as an anti-pattern. While it may be true in general that adding a new team called "DevOps" is an anti-pattern, this doesn't stop it from working in some cases. The problems lead to this being an anti-pattern isn't the structure itself, but the cultural motivation for the structure.

Some common DevOps structural anti-patters include:

- Rebranding teams to add the word "DevOps"
- Creating a new DevOps team that is separate to development and operations
- Having developers expand their skills to simply cover everything

In general cases, these are indeed anti-patterns, but only when they result in the behavior you would guess they would promote.

You wouldn't expect rebranding teams would work if everything else remains the same, but it *is* true that team identity contributes to changes in culture and interactions.

You'd imagine that adding another team between development and operations would only move them further apart, when we hoped we would be bringing them closer together, but it's possible this team will spread positive improvements and foster collaboration.

You might think that making developers responsible for running their own code would create too many distractions, stretch people's skills too much, or result in burnout. Yet many teams are reporting success from a "you build it, you run it" approach, because they have the right mix of skills and techniques to do this well.

If an organization is making changes that go against the grain of common opinion, you can only judge on it on its long-term success. 


INTRO THIS BIT


### You built it, you run it

For example, in a team with high-caliber individuals, it is absolutely possible for one team to develop, deploy, and operate their own application without depending on any external team. Such a team would be composed of diverse skill sets with lots of overlap on the core needs of the team. For example, most of the team can code, but each team member brings additional skills such as automation, quality assurance, security, or operations.

To manage such a team, you need to pay careful attention to the balance of skills and work, but the outcome is that people outside of the organization believe the team is far bigger than it is, because they can deliver high value rapidly and safely.

The single cross-functional team approach has a number of properties:

- The goals of the team is naturally aligned
- There are no external dependencies
- The team culture and identity is strong


- Skill load needs careful attention
- The architecture needs strong team alignment
- Hiring new team members is more challenging

### Site Reliability Engineering (SRE)

Site Reliability Engineering pre-dates DevOps and has been in use at Google since shortly after 2003. They describe it as "what happens when you treat operations as if it's a software problem" and they reference Margaret Hamilton work on the Apollo Project as the first great example of the discipline.

A Site Reliability Engineering team has traditional operations responsibilities, such as keeping revenue-critical systems running and secure, but due to the scale of the operation, an engineering approach is required to manage the scale of the problem.

Google hires SREs based on a combination of software engineering and operations skills, so either developers with operations knowledge, or system admins with programming skills.

Because companies such as Google have created open-source tools in the SRE space, companies with smaller-scale operations are able to apply these tools in their own SRE endeavor. This means the economics are constantly shifting in favor of SRE practices at smaller scales. While a company with billions of users has the scale to invest in innovating their own tools to manage Site Reliability Engineering, organizations with thousands of users can re-use many of these tools to automate more of their own operations tasks.

standardization and automation

Importantly, Site Reliability Engineering manages availability using service level objectives that anticipate disruptions with error budgets. The service level objectives are used to encourage other systems to be robust to outages.

Google recognize that Site Reliability Engineering is compatible with DevOps. SREs can be embedded within teams, or part of a dedicated SRE team.

### Platform engineering

Platform Engineering is well-aligned to DevOps. An internal platform team is spun around 180 degrees to put the developers in the customer seat. The platform team creates an internal platform that reduces the operations and automation burden on the developers. The platform team provides low-friction pathways for developers to get a working environment and make it easy to do the right thing in terms of automation, persistence, security, deployments, monitoring, logging, and infrastructure.

As your externally facing platform increases in size, platform engineering is there to flatten the complexity curve while preserving the autonomy of individual teams. Without a platform team it is common for teams to diverge rapidly in how they solve non-core problems. Across 20 teams, you can burn thousands of hours on concerns that could instead be part of the platform, such as how you log exceptions and messages, or how you collect metrics for monitoring. A platform team is a way to provide standards-as-a-service without dictating technology and implementation details to teams.

The skills of the platform engineering team are difficult to recruit for, but it is easier to find a platform engineering team's worth of these folks than it is to embed two similar people in each team. The platform engineering team also means total focus on smoothing out the tricky parts of software delivery, rather than having potential distractions from other feature-team work.

A platform engineering team reduces the operations burden on all other development teams, which means you need to find fewer people with a passion for solving these problems. They don't process requests, such as "create a new test database", they provide a simple mechanism that allows a team to self-serve this request safely and securely. They should keep in regular conversation with the teams using their platform and work to solve the issues slowing those teams down.

Puppet - % of organizations using internal platforms
- Low performers: 8%
- Mid-level performers: 25%
- High-performers: 48%

This leads us to an interesting aside, related to the research into software delivery. While you can see from these numbers that internal platforms are predictive indicators of performance, you also have to acknowledge 2 key facts:

1. 8% of low performers are using internal platforms and it's not working
2. 52% of high performers aren't using internal platforms, yet are working well

Platform development teams provide a way to scale software delivery to many teams, without losing "small-team" benefits. The key is to ensure the platform development team solves problems and reduces friction, rather than becoming a gate-keeping authority.



## Different paths to the same destination

Many of the structures and practices that seem to conflict with DevOps are just different ways to reach the same place. They often refer to a different part of the whole, but the goals remain the same.

DevOps

- Measure the performance of the whole system
- Shorten and amplify feedback loops
- Create a culture of continuous learning and improvement

Site reliability engineering

- x

Platform engineering

- Smooth the development experience
- Create tools and workflows that enable self-service
- Make it easy for developers to achieve system quality attributes (such as performance, observability, and security)





There isn't a conflict between these different approaches. They are each appropriate in the right circumstances and can even compliment each other.

The key to making good choices is being context-aware. If you aren't solving inter-continental real-time systems to billions of users, copying Google won't work. Different industry, scale, and product combinations need different solutions. You wouldn't want want an organization to develop an emergency medical triage system in the same way someone develops a music streaming service, equally you wouldn't like the cost of a music streaming service that copied the software delivery practices of the medical company.

- The scale of your system and it's architecture
- The type and volume of users
- The industry and its safety needs
- The scale of your software delivery team
- The skills of your team
- The specific challenges your organization has with software delivery



Don't you want your best developers writing code, not working on operations? That depends doesn't it... if individuals are self-selecting additional responsibilities you are providing learning and growth, which is part of your role as a manager. If they are making more impact by automating infrastructure and providing a smooth experience for a number of other developers, it doesn't matter that they aren't writing as many features - they are amplifying everyone else.

This may change as the situation evolves. You might grow the team, or the product might gain significant volumes or types of customers, or your competitors may come biting at your heels, or you might lose someone from your team who was driving a particular team design and it can't work without them. Your continuous improvement process should identify these changes and trigger an adaptation of how you work.

## DevOps is dead, except it isn't

There are some reports that DevOps is dead because developers don't want to deal with infrastructure, or because of the policy and cost constraints faced by growing organizations. These are challenges you will face if you are following a "you build it, you run it" approach, unless you observe and adapt as you grow.

Site reliability engineering complements DevOps by providing one way you can solve this type of problem. Platform engineering is another way to guide your organization through these complexities. If you dropped DevOps and focused only on one of these, many other problems would stop being solved and some of these are bigger predictors of organizational performance, like culture.

DevOps includes a set of capabilities that include many that you would expect from your platform engineers and SREs (such as database change management, or monitoring and observability). But it also has many capabilities that fit outside of software delivery, like transformational leadership and lean product development, which are not part of these other approaches.

## Conclusion

You should consider site reliability engineering and platform engineering as part of your DevOps wider DevOps adoption. As all improvements ultimately need to be judged by their impact on the whole system, it isn't possible to prove the value of these activities without joining them to the other DevOps capabilities.

As you grow your software delivery team, the complexity that comes with scale will need to be managed. Some organizations will limit complexity by limiting how much freedom teams have to make choices, but site reliability engineering and platform engineering provide a mechanism that manages complexity while preserving team autonomy.

## Further reading

- Site Reliability Engineering - Betsy Byers, et al
- Building Secure and Reliable Systems - Heather Adkins, et al


Happy deployments!


## Cuts



DevOps... does it mean

1. You build it, you run it... get the developers to do the whole thing and fire your ops team
2. A new team sat between Development and Operations called something like SREs, Platform Engineering, or just "DevOps"

Or does it just mean recognizing that the traditional development and operations silos contains a built-in conflict that is smothering your organization's software delivery capability?

The issue with development and operations has been that the performance measures for the two teams brings them into direct conflict with each other. Developers are expected to deliver value faster, and operations are expected to keep everything stable. When the two teams act in isolation, their goals bring them into conflict.

The operations team doesn't want more frequent deployments, as deployments bring the risk of downtime and defeats their goal of keeping systems available.

To align the goals of the teams, DevOps encourages a change to this system of opposition and the research shows that when developers and operations teams work towards the same goal, you can reduce lead times, deploy more often, have less downtime, and be more secure.



As many years of research has proved, culture may well be the biggest predictor of software delivery performance and no amount of Agile, lean, DevOps, or 

In response to the design patterns, a series of anti-patterns was devised in 

What this looks like in practice can vary to a high degree. What looks wrong from an outside perspective may actually be the best available mode of operation given the constraints.

A cautionary tale of anti-patterns and predictive indicators...

Anti-patterns are very general descriptions, as an industry they amuse and delight us, but we must take care to understand what makes a pattern "anti". For example, you might find an anti-pattern named "Ops Embedded in Dev Team"... but the anti-pattern is where operations is not valued by the organization and operations tasks are treated in the same way as features, or as interruptions to feature development. In a high-performing team, operations is a first-class citizen.





### Separate teams with shared goals

Some organizations resolve the traditional development/operations conflict by re-aligning goals and creating the right environment for the teams to work more collaboratively. You still have development teams and operations teams, but instead of their interests being in opposition, they work together to deliver high quality software frequently and safely.