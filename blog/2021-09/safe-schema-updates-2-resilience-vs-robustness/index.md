---
title: Safe schema updates - Resilient vs robust IT systems
description: Learn about the nature of failure in complex IT systems, and the benefits and drawbacks of designing for resilience vs robustness.
author: alex.yates@dlmconsultants.com
visibility: public
published: 2021-09-13-1400
metaImage: blogimage-resilientvsrobustitsystems-2021.png
bannerImage: blogimage-resilientvsrobustitsystems-2021.png
bannerImageAlt: three database servers with a person pressing a green lock icon and person with laptop plus failure icon and green tick icon
isFeatured: false
tags:
 - DevOps
 - Database Deployments
 - Deployment Patterns
 - Testing
---
 
This post is part 2 of my safe schema updates series. 

Links to the other posts in this series are available below:

!include <safe-schema-updates-posts>

In part 1, we reviewed the common downward spiral associated with traditional attitudes toward database administration and design. Over the next few posts, we'll explore a few important theoretical concepts that help explain why it’s often the most risk-averse organizations who create the most dangerous databases. We’ll also imagine what a safer data architecture and development culture might look like.

After we've armed ourselves with a deeper understanding about why some systems are systematically more reliable than others, we'll discuss a few technical changes that teams can make, that should lead to significantly better database reliability and improved business outcomes, as well as more humane working conditions.

In this post we review the concept of resilience vs robustness within software systems. Here's my favorite short articulation of the difference: 

:::hint  
 “There has been much discussion over the past decade about building resilient systems that have three specific traits:
 - Low MTTR *[Mean Time To Recovery]* due to automated remediation to well-monitored failure scenarios.
 - Low impact during failures due to distributed and redundant environments.
 - The ability to treat failure as a normal scenario in the system, ensuring that automated and manual remediation is well documented, solidly engineered, practiced, and integrated into normal day-to-day operations.
 
Note that there is not a focus on eliminating failures. Systems without failures, although robust, become brittle and fragile. When failures occur, it is more likely that the teams responding will be unprepared, and this could dramatically increase the impact of the incident. Additionally, reliable but fragile systems can lead users to expect greater reliability than the SLO *[Service Level Objective]* indicates and for which the service has been engineered. This means that even if an SLO has not been violated, customers might be quite upset when an outage does occur.”

*From  [Database Reliability Engineering](https://octopus.com/blog/devops-reading-list#dre) by Laine Campbell and Charity Majors.*
:::

I’m sure that any database administrators reading this can think of times where an outage has annoyed some stakeholders, even if, technically, their SLOs were never breached. (Assuming they had a well-defined SLO in the first place.)

Before digging into Campbell and Majors’ comments, it's valuable to reflect on the nature of failure within complex systems. For this, many folks within the DevOps, [Safety 2.0](https://www.england.nhs.uk/signuptosafety/wp-content/uploads/sites/16/2015/10/safety-1-safety-2-whte-papr.pdf), Site Reliability Engineering (SRE), and Database Reliability Engineering (DRE) movements look to Richard Cook’s short academic work: *[How Complex Systems Fail](https://how.complexsystems.fail/)*. 

But before we get to that, it’s necessary to distinguish between systems that are truly "complex", compared to systems which are merely "complicated".

## Complex vs complicated systems

Complicated systems are difficult to understand, but with sufficient effort and patience they can be understood and outcomes can be predicted. For example, encryption algorithms are complicated. While I know the high-level basics about how public-key cryptography works, I don’t pretend to be familiar with the specific algorithms or their source code. However, I appreciate that with enough time and technical ability it’s possible to read the documentation and to understand exactly how they encrypt and decrypt our data, in a predictable fashion.

Complex systems are different – they are unpredictable. Predicting the weather, for example, is complex. We can take various measurements and plug the numbers into supercomputers, but even with the most accurate data and algorithms, meteorological predictions are never a guarantee. Our measurements and our algorithms are only close approximations, and there are elements that we still don't fully understand. Hence, is it's virtually impossible to accurately predict the weather a month ahead of time. (Especially where I live - in the UK!)

By definition, any system that includes humans is complex, rather than complicated. Humanity is yet to create a system that can accurately predict human decisions. Free will is either hard, or impossible, to model computationally – and already I feel like I’m falling down a philosophical rabbit hole. My point is, if your IT system relies on humans to maintain, update, or fix it, your IT system is complex by definition. This is because humans are a necessary part of your system, and humans are complex creatures.

And that’s before we even consider the many poorly documented or understood dependencies that we covered in [part 1](https://octopus.com/blog/safe-schema-updates-1-delivery-hell). Any system without 100% accurate and up to date documentation is (by definition) complex, because it’s impossible to truly understand and predict the consequences of a single change. Since I’m yet to come across a perfectly documented large-scale IT system, I’m yet to witness one that can be classified as complicated, rather than complex.

## Appreciating the nature of failure in complex systems

Hopefully the title *How Complex Systems Fail* now communicates a more specific idea. Cook is not talking about predictable systems. By definition, he’s talking about systems that contain unpredictable elements, just like the majority of (and probably all) enterprise level IT systems.

This said, he was primarily writing about hospitals (where he worked), and other high-risk, complex environments, such as the aviation industry or the military. If you are an IT professional who thinks the consequences of failure are stacked high for your IT system, imagine being an airline pilot, a paratrooper, or a surgeon.

For an operation to be successful, so many things need to go right. The operating theatre needs to be prepared correctly, all the necessary equipment needs to be in the right place, at the right time, and a group of highly qualified professionals need to perform their duties as a team, to a high standard, under stressful conditions, often solving unexpected problems, and making literal life and death decisions as they go.

When it doesn’t go well, I imagine the phrase “blameless post-mortem” feels especially poignant. And I expect it may be an especially important practice.

While it’s rare for IT failures to result in the loss of life, any experienced IT professional would read *How Complex Systems Fail* and instinctively understand that it applies to enterprise IT just as it applies to any hospital, airliner, or battleship. 

*[How Complex Systems Fail](https://how.complexsystems.fail/)* is roughly a 10-minute read and, in my opinion, it should be mandatory reading for any computer science student or IT professional. In the article, Cook highlights 18 specific and measurable attributes that are common in mature complex systems:

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

Remember, these are specifically in reference to *unpredictable* complex systems, like the hypothetical database and its dependencies from [the previous post](https://octopus.com/blog/safe-schema-updates-1-delivery-hell).

Take a breath, then read that list again. Those observations are important for what comes next.

I’m not going to defend or justify any of those statements. Nor am I going to expand on them any further – Cook does that perfectly well for himself and I wouldn’t add much value by repeating the exercise. I am going to accept them and assume they are true. If you are not ready to take that leap, I suggest you read *[How Complex Systems Fail](https://how.complexsystems.fail/)* in full before continuing.

What follows is a discussion about how we can and should respond to these observations about the nature of failure in our IT systems.

## Why resilient IT systems are safer than robust IT systems

The DevOps movement is often thought of as the evolution of Lean ideas born from the Japanese car manufacturing industry in the 1970s and '80s. Arguably, however, especially in more recent years, DevOps owes as much heritage to the Safety 2.0 movement, pioneered by Cook and his contemporaries. 

Rather than automobile manufacturing and supply chain management, Safety 2.0 was developed in the health care sector in the '90s and '00s. Safety 2.0 is a management philosophy designed to nurture a 'safety culture' in the sorts of complex systems that Cook described in *How Complex Systems Fail*. 

Time for another definition, shamelessly lifted from excellent work by other people:

:::hint 
Most people think of safety as the absence of accidents and incidents (or as an acceptable level of risk). In this perspective, which we term Safety-I, safety is defined as a state where as few things as possible go wrong. A Safety-I approach presumes that things go wrong because of identifiable failures or malfunctions of specific components: technology, procedures, the human workers and the organizations in which they are embedded. Humans — acting alone or collectively — are therefore viewed predominantly as a liability or hazard, principally because they are the most variable of these components. The purpose of accident investigation in Safety-I is to identify the causes and contributory factors of adverse outcomes, while risk assessment aims to determine their likelihood. The safety management principle is to respond when something happens or is categorized as an unacceptable risk, usually by trying to eliminate causes or improve barriers, or both.
 
[...]
 
Crucially, the Safety-I view does not stop to consider why human performance practically always goes right. Things do not go right because people behave as they are supposed to, but because people can and do adjust what they do to match the conditions of work. As systems continue to develop and introduce more complexity, these adjustments become increasingly important to maintain acceptable performance. The challenge for safety improvement is therefore to understand these adjustments — in other words, to understand how performance usually goes right in spite of the uncertainties, ambiguities, and goal conflicts that pervade complex work situations. Despite the obvious importance of things going right, traditional safety management has paid little attention to this.

Safety management should therefore move from ensuring that ‘as few things as possible go wrong’ to ensuring that ‘as many things as possible go right’. We call this perspective Safety-II; it relates to the system’s ability to succeed under varying conditions. A Safety-II approach assumes that everyday performance variability provides the adaptations that are needed to respond to varying conditions, and hence is the reason why things go right. Humans are consequently seen as a resource necessary for system flexibility and resilience. In Safety-II the purpose of investigations changes to become an understanding of how things usually go right, since that is the basis for explaining how things occasionally go wrong. Risk assessment tries to understand the conditions where performance variability can become difficult or impossible to monitor and control. The safety management principle is to facilitate everyday work, to anticipate developments and events, and to maintain the adaptive capacity to respond effectively to the inevitable surprises (Finkel 2011).
 
Hollnagel E., Wears R.L. and Braithwaite J. From Safety-I to Safety-II: A White Paper. The Resilient Health Care Net: Published simultaneously by the University of Southern Denmark, University of Florida, USA, and Macquarie University, Australia.
[Available online here.](https://www.england.nhs.uk/signuptosafety/wp-content/uploads/sites/16/2015/10/safety-1-safety-2-whte-papr.pdf)
:::

The biggest lesson I take from this, is that safety is something you actively build and refine, rather than a gatekeeper who catches mistakes. It's healthier to add systems that create safety, than it is to try to catch all the mistakes. After all, given enough time, it's hopelessly unrealistic to expect that you'll catch them all.

## Real world examples of resilient IT systems

Practically, what does that look like? Well, it could look like many things. Netflix is often painted as the poster-child for resilience engineering in IT, so let's discuss what they do.

On 21st April 2011, [AWS suffered a major outage in their Virginia (US-East-1) region](https://aws.amazon.com/message/65648/). This failure took down many major sites including Reddit, Hootsuite, Quora and the Windows Facebook App. Netflix, however, weathered the storm. Their users barely noticed.

Afterwards, [Netflix shared a blog post](https://netflixtechblog.com/lessons-netflix-learned-from-the-aws-outage-deefe5fd0c04) that explained some of the techniques they used which allowed them to stay online when so many others failed. And, by the way, 2011 wasn’t a one off. Netflix did it again in [2015](https://www.techrepublic.com/article/aws-outage-how-netflix-weathered-the-storm-by-preparing-for-the-worst/), and again in [2017](https://www.networkworld.com/article/3178076/why-netflix-didnt-sink-when-amazon-s3-went-down.html). They now possess a reputation for producing one of the safest and most resilient systems on the web.

Why was Netflix uniquely capable of surviving the storm? In their own words (from the blog above), their “systems are designed explicitly for these sorts of failures”. They recognize what Cook has taught us: that *complex systems are intrinsically hazardous systems* and that *catastrophe is always just around the corner*, and they design their systems with these facts in mind.

Their *systems are heavily defended against failure*, with many clever automated failover and redundancy characteristics. They also explicitly design ways to *run in degraded mode*. For example, they recognize that providing users with recommendations is valuable, but not essential, and they are aware that it’s computationally expensive. Hence, when the system is struggling, they can automatically turn that (and many other ‘nice-to-have’ features) off in order to keep core operations online.

Fundamentally, Netflix, as well as any other resilient systems, recognize that at any one time they will *contain changing mixtures of failures latent within them*. Rather than designing solely robust systems which focus on the impossible task of avoiding failure, they recognize that all systems and dependencies can fail without notice and in unpredictable ways. Hence, it’s paramount that all systems are designed to stay online, albeit perhaps in a degraded mode, even in the face of one or more failures, whether they be internal infrastructure/code failures, or dependencies.

What’s more, since *failure free operations require experience with failure*, it’s necessary to practice failures. And it’s necessary to practice them at inconvenient times. That’s why Netflix intentionally break their own services at random. If you’ve never heard of “chaos monkey” or the “Simian Army”, you should [read about it from the folks who created it](https://netflixtechblog.com/the-netflix-simian-army-16e57fbab116).

Finally, as Cook made clear, with various points, the human operators are integral parts of the system. The Netflix blog post stuck to software details, but the people need to be valued and they need to be trained, protected and kept safe too – just like any other part of the system.

What’s more, since *human practitioners are the adaptable element of complex systems*, they are the part to invest in. Most importantly, we explicitly recognize that *all practitioner actions are gambles*, *hindsight biases post-accident assessments of human performance*, and that *post-accident attribution of a ‘root cause’ is fundamentally wrong*. Hence, blame and scapegoating of specific individuals has no place in the development of safe IT systems. Where humans have made mistakes, they are offered training and support, rather than performance management or severance packages. If one person can make this mistake, others can surely repeat it. Rather than trying to remove the individual from the system, safety measures should be added to the system to protect individuals from repeating similar mistakes in the future. After all, since *failure free operations require experience with failure*, that individual is likely to have learned a valuable lesson and may be uniquely placed to contribute toward the development of such a test/check.

As Tom Watson, CEO of IBM (1956-71) articulated it, when a young executive asked if he was going to be fired following a costly mistake: *"Not at all, young man, we have just spent a couple of million dollars educating you."*

## Practically, what does this mean for my database?

First, we need to recognize that *change introduces new forms of failure*. That’s not to say that we should stop making changes. If anything we should be making changes more often! But we should be designing our change process such that those changes are tested as effectively as possible, and that they can be reverted easily when errors do slip through. That’s a uniquely challenging task with databases, and we’ll talk about it more in the next post about Continuous Integration, as well as towards the end of this series when we discuss patterns for provisioning environments and near-zero downtime deployments.

We should also be designing effective fire breaks. One bad database update should not be causing cascading failures. Failures need to be contained so that their impact is minimized. This means we need to be avoiding monolithic shared databases and we should be trying to split them up into smaller, simpler systems that are capable of running independently, even if they need to temporarily run in some degraded capacity. We’ll look more closely at that in the later posts about loose coupling and the Strangler Pattern.

The outcome of these changes should be significantly reduced risk associated with any database update. This reduced risk should reduce the need for overly bureaucratic change management processes, allowing the more frequent delivery of smaller, safer updates, reversing the downward spiral discussed in the last post, and leading to a period of continuous improvement.

## Next time

In the next post (part 3) we’ll talk about Continuous Integration (CI). Specifically, we’ll discuss how it’s been misunderstood and the harm that misunderstanding can cause, as well as the benefits of embracing “proper” CI for any IT system – including any relational database. 

Links to the other posts in this series are available below.

!include <safe-schema-updates-posts>

## Watch the webinars 

Our first webinar discussed how loosely coupled architectures lead to maintainability, innovation, and safety. Part two discussed how to transition a mature system from one architecture to another. 

### Database DevOps: Imagining better systems

<iframe width="560" height="315" src="https://www.youtube.com/embed/oJAbUMZ6bQY" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### Database DevOps: Building better systems

<iframe width="560" height="315" src="https://www.youtube.com/embed/joogIAcqMYo" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Happy deployments!
