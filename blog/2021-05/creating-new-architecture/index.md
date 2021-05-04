---
title: Creating New Architecture
description: Creating a new architecture can be a daunting challenge. This post will look at how we approach creating new architecture within Octopus, and give some real examples on how to tame complexity and achieve success.
author: Andrew Best
visibility: private
published: 3020-01-01
metaImage:
bannerImage:
tags:
  - architecture
---

Within engineering organisations that build large software products, complexity can often grow at a hard-to-measure pace. This complexity can be static and measurable - as you add more functionality and relationships within your software, its static complexity grows. It can also be emergent - as your software gains more functionality and users, those users will find more unique uses for it, many of which you will not have anticipated!

Over time, these complexities make changing your software harder. Engineers struggle to attain mastery across the software as it becomes larger and more complex. Things take a lot longer to do as the cognitive overhead of implementing any change keeps growing. Keeping quality up takes more effort as there are more things that need to be taken into account with every change.

It can be challenging enought to identify you are _in_ that scenario. However identifying it is critical, as it usually takes systemic change to get _out_ of it.

Octopus Deploy is a large and complex software product. We have some ambitious plans to execute on through 2021 and onwards. However we are not immune to the challenges of developing high quality, large scale software solutions. Our engineering leadership team had been monitoring some common recurring themes throughout 2020. It _was_ getting harder to understand Octopus end-to-end as an engineer - which is understandable, it has an incredible amount of functionality, and supports a very wide variety of organisation structures, platforms, and deployment requirements. It was taking longer to implement changes too, as code changes could potentially impact many other parts of the growing codebase.

Knowing that it was becoming more challenging to ship high quality changes to Octopus with speed, and understanding we had some ambitious plans going forward, we established an engineering strategy for 2021 that put these chalenges front-and-center of our focus.

Part of the strategy involves creating separate value streams within Octopus - teams that can make decisions independently and move with speed to ship valuable changes within Octopus to our customers.

One of those value streams focuses on Steps - the things that do the work of deployment within Octopus.

We wanted to create an entirely now architecture for steps, one that would allow us to quickly deliver new value to our customers, and to rapidly establish a high-performing team responsible for delivering them.

## Architecture

In software, architecture encompasses the structure of the system, the relationships between its components, the properties the system emits (sometimes referred to as the "ilities"), and the behaviours the system encodes. Some systems may comprise a single, large piece of software. Others may be decomposed into smaller sub-systems that work together to accomplish goals.

The essence of good architecture is that it helps you make decisions and changes with speed and confidence. Want to add a new behaviour? Good architecture will make it easy to reason about doing that, and to execute on it. Poor architecture will have you reasoning about how the world came to be before you can write a line of code. Carl Sagan once said "If you wish to make apple pie from scratch, you must first create the universe.". You _don't_ want to have to perform that sort of reasoning in software - you just want to have to reason your way up from a good set of apple pie ingredients.

Making changes with speed and confidence is a must at Octopus. The landscape of release management is growing at an unprecedented rate. New technology ecosystems are being created in which workloads can be created. New cloud service are arriving all of the time, providiong new, novel ways to host workloads. Octopus' goal is to make shipping those workloads onto those new platforms world-class, and we want to enable those experiences in Octopus as rapidly as possible.

To ensure we can achieve that goal, at Octopus we have been investing time and effort into creating a brand new architecture for developing Steps within Octopus - the things that do the work of deployment!

Creating a new architecture is always an interesting challenge - where do you start? What problems do you need to solve? How do you actually deliver something and not get lost down endless rabbit holes? In this article we will explore how to approach defining and building a new software architecture in a way that increases your chances of succeess.

### Goals

Success in software comes from knowing where the finish line is in any endeavour. How will you know if you have succeeded? Success isn't just having some new software. This holds true if you are defining a whole new architecture, or just building a small feature. If you don't know where you are going, any road will get you there - and we don't want to arrive at just anywhere, we want to succeed!

After initial collaboration with our primary stakeholders, and with Paul, our CEO, we agreed on the following sets of goals for the new steps architecture:

- Steps should be able to be shipped out-of-band of Octopus Server releases
- Steps should be simple and easy to develop
- Steps should be able to be developed in a technology other than C#/.NET

These goals underpin Octopus' business goals of building up a high-performing team to deliver new deployment capabilities, and to be able to more rapidly enable new deployment scenarios for Octopus' customers.

Goals are a must-have as they help you reason about decisions, and avoid rabbit holes. They provide a great "litmus test" when deciding whether you should do something one way, or another. If something takes you toward one of the goals, it is probably a good thing. If it leads you away from a goal, it might be a bad thing. If it doesn't contribute to a goal at all, it is probably not needed at all.

### Constraints

Once we have an agreed set of goals that define what success is, we can get into defining the architecture itself. Once again, architecture refers to a lot of things:

> architecture encompasses the structure of the system, the relationships between its components, the properties the system emits (sometimes referred to as the "ilities"), and the behaviours the system encodes

For steps, we started by defining structures - breaking down what "steps" actually are into sub-systems so that we could reason and make decisions about them.

These included:

- The Step UI
- The Step Executor
- Inputs and Outputs
- The Programming Model
- Server Integration (The packaging model)
- Deployment Execution

Initial conversations can be challenging when defining new architecture - you'll find that at times it can be hard to "land the plane" on conversations, as you will circle around how a decision in one sub-system will impact the others, exploring different viewpoints such as user experience, developer experience, and product experience with each lap. One line of conversation will take you on a full-circle lap of sub-systems, until you arrive again at the first point you were trying to decide on.

If you find you are struggling to make decisions early on, and conversations feel circular or never-ending, it is likely you are missing a key ingredient - constraints.

Constraints limit the freedom of choice we have when defining and developing our architecture.

This is exactly what we want early on - we want to make strong decisions about constraints we can impose, as it will limit our choices, and this makes decisions easier.

Some of our goals already impose strict constraints - "steps needed to be able to be developed in a technology other than .NET" gives us a pretty clear initial constraint.

Others were not as strict - "simple and easy to develop" provides context, but is not a hard-and-fast constraint - it is subjective.

In the early stages of designing our new architecture, we were struggling to gain clarity and concensus around the programming and composition model for steps. If a user wanted a step to behave "just a bit differently", and re-order the logical inner sequencing of the steps behaviour, or perhaps inject some of their own unique behaviour within the step - would that be something we want to support?

By taking a stand on the above potential requirement, we would be able to establish a clear constraint. This constraint would impact many of he architecture's sub-systems - the UI, the executor, the programming model, it even ventured further outside of our initial boundary and would potentially impact other areas of Octopus.

Working together, we decided that we wouldn't need to enable arbitrary composition within steps - if you pursue this line of thinking, eventually you'd end up needing to produce a DSL or programming language to build up deployment processes, as that would be the only thing flexible enough to satisfy all use cases! Instead of pursuing that angle, we instead made a decision that we would focus on providing high-leverage, high-value steps to our users, and give users a frictionless way to progress from opinionated steps to more flexible steps (run a template, run a script / cli) should they require their own unique behaviours.

This constraint had an immediate impact - we could reason more clearly about our UI, our executors, our programming model, and the impact on Octopus itself, thanks to the clear limitations this constraint imposed.

### Decision Making

When making decisions at Octopus, we are strong believers in creating concensus, and then executing strongly on the basis of it. Almost all impactful decisions are deeply scruitinized. When defining architecture, this scruitiny allows you to better forsee system-level impacts of architectural choices. This is one of the _hardest_ things to do when building new architecture, but it is also the _most important_.

To make great decisions, you need to ensure the right experts have had inputs into your architectural decisions.

This does not mean design-by-committee - ownership is important, and you as an architect should own the architecture you develop. However it mean you have the responsibility to widely seek input into your architectural designs - finding experts in other sub-sytems outside of your sphere of control that may need to influence your designs.

One example of this with our new architecture was an overlap identified with Project Bento - our brand new project import/export system. By talking with the team developing Bento, we discovered that there was a shared piece of the system under both of our initiatives - the input model for steps. Bento needed to know if a given set of inputs contained an Account, or other domain-specific resources within Octopus. It would use this knowledge to "crawl" the set of resources it would need to export/import across spaces. We were proposing to redefine how inputs were modelled within Octopus. We needed to make sure our proposed architecture in this space would still satisfy Bento's requirements.

Another thing that can assist decision making is keeping our goals front-of-mind. Whilst our constraints limit our choices, so we know what things we don't need to make decisions about, goals help us decide between multiple potentially valid options.

Goals can be used as a litmus test. Does this decision take us toward achieving this goal, or does it push us further away from acheiving it? Ensure they are considered for all fundamental decisions. We have revisited our goal of steps being "simple and easy to develop" numerous times when deciding how to implement the various APIs that underpin the new architecture.

### Complexity

Complexity within architecture tends to come in two categories - static complexity, which deals with the systems components and their relationships, and emergent complexity, which comes from users using your software in novel and unique ways.

Making architectural decisions needs to take both into account.

Static complexity tends to impact sensemaking - it is hard to make a decision if the area you are working within is very complex. It can be hard to reason about all the ways your decision might impact various sub-systems.

The solution to this problem is getting stuck into analysis. We use [Whimsical](https://whimsical.com/) heavily at Octopus, but many other diagramming tools can be of great assistance in these scenarios. Something that allows you to build flowcharts and visualise connections between various sub-systems in specific contexts will help you find all of the places you need to consider when making a decision. There is no avoiding this type of analysis - if you don't do it, you'll make wrong assumptions that will bite you later. It is hard to "outsource" this type of analysis - someone might be able to describe to you the inner workings of a particular sub-system, but if they don't have a detailled map to provide you, it is likely you're going to need to pull the code and get diagramming.

Emergent complexity comes with attempting to anticipate how humans may interact with the system once built, or how the usage of the system may change over time, and how you will need to accommodate that change.

Emergent complexity can be dealt with in a couple of ways.

Firstly, we can go back to constraints. Can we constrain the ways we will enable users to use our system? This will limit the emergent complexity that is possible, and simplify our decision making.

If we have constraints in place, we can then look at how certain implementation decisions may increase or decrease emergent complexity.

When we were deciding how UI should be expressed for steps, we were faced with a decision. Should we let users bring their own HTML, javascript, and framework to express the step UI? What about just some html? What if it were more of a code-based DSL? What about just plain old declarative JSON?

There was a particular class of emergent complexity we wanted to avoid - the impact of Octopus UI changes across hundreds or thousands of steps, should those changes be necessitated in the future. By acknowleding this emergent complexity, we were able to make a decision that limited it - we decided to implement a DSL that could be used to express a step's UI - this would give people the power and flexibility of implementing the UI in code, and would avoid the complexity that would come with people supplying arbitrary HTML and javascript.

### "ilities"

Ilities, or [system quality attributes](https://en.wikipedia.org/wiki/List_of_system_quality_attributes), refer to non-functional requirements that a system may need to adhere to.

A good architecture will emit properties that support the "ilities" that you have identified as important. These "ilities" tend to cut across all of the sub-systems within an architecture.

An example of an "ility" that is high on our mind for the new steps architecture is maintainability.

If we need to make changes on one side of an interface boundary between steps and Octopus itself, we want to make sure we don't need to force the propagation of that change across hundreds of steps (or require step authors to do the same).

Knowing this is something that is important to our architecture, we are focussing with great detail on our API surfaces which form the interfaces between steps and Octopus, on versioning, and on compatibility.

There are many compatibility surfaces within the steps architecture. Making sure these surfaces have explicit versioning in-place will allow us to make changes of them over time, and make deliberate decisions about their compatibility as they evolve.

It is important to brainstorm the various "ilities" that might be important to your architecture early on, so that you can take them into account as your architecture evolves.

## Conclusion

Good software architecture allows you to tame the complexity of software as it grows, enabling you to develop new functionality, or change existing functionality, with speed and confidence.

Good architecture:

- Is founded on a set of clear goals that can be tied to business goals
- Expresses strong constraints that limit the complexity the architecture needs to support
- Is developed on the back of many high-quality decisions, which have had the appropriate analysis, scruitiny, and expert input applied
- Acknowledges complexity, by ensuring it is understood with deep analysis, and designed for when it is emergent
- Addresses the important "ilities", making sure they are considered and designed for within all of the sub-systems defined within the architecture
