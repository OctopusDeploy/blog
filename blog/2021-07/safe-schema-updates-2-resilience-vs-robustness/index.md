---
title: Safe Schema Updates - Resilient vs Robust IT Systems
description: TODO description
author: alex.yates@dlmconsultants.com
visibility: private
published: 2022-07-28-1400
tags:
 - DevOps
 - Database Deployments
 - Deployment Patterns
 - Testing
---

This blog post is part 2 of my Safe Schema Updates series. The other posts in this series are available at the following links:

!include <safe-schema-updates-posts>

In part 1 of this series, we reviewed the common death-spiral associated with traditional attitudes toward database administration and design. Over the next few posts, we’ll explore a few important theoretical concepts that go some-way to explaining why it’s often the most risk-averse organisations who create the most dangerous databases, and we’ll imagine what a safer data architecture and development culture might look like.

Only once we’ve armed ourselves with a deeper understanding about why some systems are so much more reliable than others, will we discuss a few technical changes that teams can make that should lead to significantly better database reliability and improved business outcomes, as well as more humane working conditions.

In this post we’ll review the concept of resilience vs robustness within software systems. We’ll start with my favourite short articulation of the difference: 

<blockquote>
*“There has been much discussion over the past decade about building resilient systems that have three specific traits:
- Low MTTR *[Mean Time To Recovery]* due to automated remediation to well-monitored failure scenarios.
- Low impact during failures due to distributed and redundant environments.
- The ability to treat failure as a normal scenario in the system, ensuring that automated and manual remediation is well documented, solidly engineered, practiced, and integrated into normal day-to-day operations.

Note that there is not a focus on eliminating failures. Systems without failures, although robust, become brittle and fragile. When failures occur, it is more likely that the teams responding will be unprepared, and this could dramatically increase the impact of the incident. Additionally, reliable but fragile systems can lead users to expect greater reliability than the SLO *[Service Level Objective]* indicates and for which the service has been engineered. This means that even if an SLO has not been violated, customers might be quite upset when an outage does occur.”

From  Database Reliability Engineering by Laine Campbell and Charity Majors.*
</blockquote>

I’m sure that any database administrators reading this can think of times where an outage has annoyed some stakeholders, even if, technically, their SLOs were never breached. (Assuming they even had a well defined SLO.)

Before digging into Campbell and Majors’ comments, it would be valuable to reflect on the nature of failure within complex systems. For this, many folks within the DevOps, Safety 2.0, Site Reliability Engineering (SRE), and Database Reliability Engineering (DRE) movements look to Richard Cook’s short academic work: *How Complex Systems Fail*. 

But before we get to that, it’s necessary to distinguish between complicated systems, and complex systems.

Complex vs Complicated Systems

Complicated systems are difficult to understand, but with sufficient effort and patience they can be understood and outcomes can be predicted. For example, encryption algorithms are complicated. I don’t pretend to know the fine details about how the algorithms work, but I understand that they do, and I appreciate that with enough time and technical ability it’s possible to read the documentation and to understand exactly how they encrypt and decrypt our data, in a predictable fashion.

Complex systems are different – they are unpredictable. Predicting the weather, with current technology, for example, is complex. We can take various measurements and plug various statistics into supercomputers, but even with the most accurate data and algorithms, meteorological predictions are never a guarantee because our measurements and our algorithms are only close approximations, and we don’t yet fully understand all the factors involved. Trying to accurately predict the weather a month ahead of time is virtually impossible. (Especially where I live - in the UK!)

By definition, any system that includes humans is complex, rather than complicated. Humanity is yet to create a system that can accurately predict human decisions. Free will is either hard, or impossible, to model computationally – and already I feel like I’m falling down a philosophical rabbit hole. My point is that, if your IT system relies on humans to maintain, update or fix it, your IT system is complex by definition. This is because humans are a necessary part of your system, and humans are complex creatures.

And that’s before we even consider the myriad of poorly documented or understood dependencies that we covered in part 1. Any system without 100% accurate and up to date documentation is (by definition) complex, because it’s impossible to truly understand and predict the consequences of a single change. Since I’m yet to come across a perfectly documented large-scale IT system, I’m yet to witness one that can be classified as complicated, rather than complex.

Appreciating the nature of failure in complex systems

Hopefully the title *How Complex Systems Fail* now communicates a more specific idea. Cook is not talking about predictable systems. By definition, he’s talking about systems that contain unpredictable elements, just like the majority of (and probably all) enterprise level IT systems.

This said, he was primarily writing about hospitals (where he worked), and other high-risk, complex environments, such as the aviation industry or the army. If you are an IT professional who thinks the consequences of failure are stacked high for your IT system, imagine being a pilot, a soldier, or a surgeon.

For an operation to be successful, so many things need to go right. The operating theatre needs to be prepared correctly, all the necessary equipment needs to be in the right place, at the right time, and a large team of highly qualified professionals need to perform their duties together at a high standard under stressful conditions, often solving unexpected problems and making literal life and death decisions as they go.

When it doesn’t go well, I imagine the phrase “blameless post-mortem” feels especially poignant. And I expect it may be an especially important practice.

While it’s rare for IT failures to result in the loss of life, any experienced IT professional would read *How Complex Systems Fail* and instinctively understand that it applies to enterprise IT just as it applies to any hospital, commercial airliner, or battleship. 

*How Complex Systems Fail* is roughly a 10-minute read and, in my personal opinion, it should be mandatory reading for any computer science student or IT professional. In it Cook highlights 18 specific and measurable attributes that are commonplace within mature complex systems:

1.	Complex systems are intrinsically hazardous systems.
1.	Complex systems are heavily and successfully defended against failure.
1.	Catastrophe requires multiple failures – single point failures are not enough.
1.	Complex systems contain changing mixtures of failures latent within them.
1.	Complex systems run in degraded mode.
1.	Catastrophe is always just around the corner.
1.	Post-accident attribution of a ‘root cause’ is fundamentally wrong.
1.	Hindsight biases post-accident assessments of human performance.
1.	Human operators have dual roles: as producers and defenders against failure.
1.	All practitioner actions are gambles.
1.	Actions at the sharp end resolve all ambiguity.
1.	Human practitioners are the adaptable element of complex systems.
1.	Human expertise in complex systems is constantly changing.
1.	Change introduces new forms of failure.
1.	Views of ‘cause’ limit the effectiveness of defenses against future events.
1.	Safety is a characteristic of systems and not their components.
1.	People continuously create safety.
1.	Failure free operations require experience with failure.

Remember, these are specifically in reference to *unpredictable* complex systems, like the hypothetical database and its dependencies from the previous post.

Take a breath, then read that list again. Those observations are important for what comes next.

I’m not going to defend or justify any of those statements. Nor am I going to expand on them any further – Cook does that perfectly well for himself and I wouldn’t add much value by repeating the exercise. I am going to accept them and assume they are true. If you are not ready to take that leap, I suggest you read *How Complex Systems Fail* in full before continuing.

What follows is a discussion about how we can and should respond to these observations about the nature of failure in our IT systems.

Why resilient IT systems are safer than robust IT systems

On 21st April 2011, AWS suffered a major outage in their Virginia (US-East-1) region. This failure took down many major sites including Reddit, Hootsuite, Quora and the Windows Facebook App. Netflix, however, weathered the storm. Their users barely noticed.

Afterwards, Netflix shared a blog post explaining some of the techniques they used that allowed them to stay online when so many others were taken down… And, by the way, 2011 wasn’t a one off. Netflix did it again in 2015, and again in 2017. They now possess a reputation for producing one of the safest and most resilient, systems on the web.

Why was Netflix uniquely capable of surviving the storm? In their own words (from the blog above), their “systems are designed explicitly for these sorts of failures”. They recognise what Cook has taught us: that *complex systems are intrinsically hazardous systems* and that *catastrophe is always just around the corner*, and they design their systems with these facts in mind.

Their *systems are heavily defended against failure*, with many clever automated failover and redundancy characteristics. They also explicitly design ways to *run in degraded mode*. For example, they recognise that providing users with recommendations is valuable, but not essential, and they are aware that it’s computationally expensive. Hence, when the system is struggling, they can automatically turn that (and many other ‘nice-to-have’ features) off in order to keep core operations online.

Fundamentally, Netflix, as well as any other resilient systems, recognise that at any one time they will *contain changing mixtures of failures latent within them*. Rather than designing solely robust systems which focus on the impossible task of avoiding failure, they recognise that all systems and dependencies can fail without notice and in unpredictable ways. Hence, it’s paramount that all systems are designed to stay online, albeit perhaps in a degraded mode, even in the face of one or more failures, whether they be internal infrastructure/code failures, or dependencies.

What’s more, since *failure free operations require experience with failure*, it’s necessary to practice failures. And it’s necessary to practice them at inconvenient times. That’s why Netflix intentionally break their own services at random. If you’ve never heard of “chaos monkey” or the “Simian Army”, you should read about it from the folks that created it.

Finally, as Cook made clear, with various points, the human operators are integral parts of the system. The Netflix blog post stuck to software details, but the people need to be valued and they need to be trained, protected and kept safe too – just like any other part of the system.

What’s more, since *human practitioners are the adaptable element of complex systems*, they are the part to invest in. Most importantly, we explicitly recognise that *all practitioner actions are gambles*, *hindsight biases post-accident assessments of human performance*, and that *post-accident attribution of a ‘root cause’ is fundamentally wrong*. Hence, blame and scapegoating of specific individuals has no place in the development of safe IT systems.

Practically, what does this mean for my database?

First, we need to recognise that *change introduces new forms of failure*. That’s not to say that we should stop making changes. If anything we should be making changes more often! But we should be designing our change process such that those changes are tested as effectively as possible, and that they can be reverted easily. That’s a uniquely challenging task with databases, and we’ll talk about it more in the next post about Continuous Integration, as well as towards the end of this series when we discuss patterns for provisioning environments and near-zero downtime deployments.

We should also be designing effective fire breaks. One bad database update should not be causing cascading failures. Failures need to be contained so that their impact is minimised. This means we need to be avoiding monolithic shared databases and we should be trying to split them up into smaller, simpler systems that are capable of running independently, even if they need to temporarily run in some degraded capacity. We’ll look more closely at that in the later posts about loose coupling and the Strangler Pattern.

The outcome of these changes should be significantly reduced risk associated with any database update. This reduced risk should reduce the need for any overly bureaucratic change management processes, allowing the more frequent delivery of smaller, safer updates, reversing the death spiral discussed in the last post, and leading to a period of continuous improvement.

## Next Time

In the next post (part 3) we’ll talk about Continuous Integration (CI). Specifically, we’ll discuss how it’s been misunderstood and the harm that misunderstanding can cause, as well as the benefits of embracing “proper” CI for any IT system – including any relational database. 

!include <safe-schema-updates-posts>
