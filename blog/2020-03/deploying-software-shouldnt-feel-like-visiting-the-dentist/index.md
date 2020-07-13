---
title: Deploying software shouldn't feel like visiting the dentist
description: Reasons you should deploy early and often.
author: info@red-gate.com
visibility: public
published: 2020-03-25
metaImage:
bannerImage:
canonicalurl: https://www.red-gate.com/blog/software-development/software-deployment
tags:
 - DevOps
---

As a project manager, the process of deploying software can be fraught with anxiety and dread. I can think about numerous times in my career where, in the shadow of an approaching release deadline, the thought of having to deploy our application was akin to contemplating an imminent visit to the dentist. In fact, even the mechanism for getting our ‘awesome new features’ into the hands of our users felt like pulling teeth.

The temptation is to deploy as infrequently as possible, just as the temptation is to visit the dentist as rarely as you can get away with it. You know you ought to go more regularly but you fear it’s going to be really painful. You may even have a haunting memory of a previous visit that ended up with your pearly whites on the receiving end of the dental drill. Personally, my teeth ache just thinking about it.

![](post-its.png "width=500")

Instead of visiting the dentist regularly, maybe at six monthly intervals, you might find yourself making excuses and putting it off. As a result, you never get around to that check-up and only drag yourself into the dental surgery two years later, when you’ve got a cavity the size of the Grand Canyon. I know I’ve done this, fully aware of the risk that it might happen and have subsequently chastised myself when it does. Ironically, the resultant marathon stint in the dentist’s chair contributes to the haunting memories that compelled me to avoid check-ups in the first place.

In the software development world, leaving an extended period between deployments of your software is just as risky an approach as forgoing regular dental check-ups. The bigger the gap between deployments, the larger the amount of work – code changes, database schema modifications, configuration alterations – that have not been validated by your customers. The greater the functionality and code delta between deployments, the more likely the new release has broken something serious, unleashing a horde of pointy-fanged bugs that are going to eat your time and leave your plan for the next development iteration in tatters.

Instead of worrying about the huge list of changes you’ve built up between releases, you can feel better by simply assuming that you got your decisions right. You can assume that you made a good call regarding which new features to add, how to design the user interface so it is ‘easy to use’ and whether the user is going to like it more when it’s drawn in mauve. However, you can only prove you got that stuff right by giving it to your users. The bigger the gap between releases, the more development effort you are gambling has gone ‘right’. When the release is finally out, your users are going to tell you if you got it right through sales figures, raised bugs, direct feedback and even social media. So, if you got that decision on drawing it in mauve wrong, that’s when the pain really starts.

Back in the dentist’s chair, the longer the gap between check-ups, the more food you’ve eaten between visits and the more unchecked bacteria builds up on your teeth. You’re assuming that you are brushing your teeth right and that your reticence to floss has not impacted too much on your dental hygiene. However, if you got that wrong, that’s when things get painful.

So, it seems sensible to deploy pretty often, right? The problem is, and where the metaphor decays (sorry), is that software development teams feel that deploying is harder than visiting the dentist. Deployment processes can be a significant overhead, especially when compared with dialing a phone number and turning up at your local dental surgery. There are code reviews to undertake, test plans to execute, documentation to write, numerous environments to update and administrative tasks to complete. Development teams would be forgiven for dreading a deployment process that consumes days of their life and invariably raises difficult questions and forces uncomfortable decisions.

Really, deploying should be less onerous, easier, cheaper and quicker than going to the dentist. In order to achieve this state, software development teams need to do three things:

- Automate their deployment mechanisms
- Re-evaluate and streamline their administrative practices and processes
- Explicitly focus on delivering valuable, incremental features to their users

Doing all that at once can seem daunting, so the best thing that a software development team can do is to start to make things better ‘now’. Even if they can only automate a small part of their deployment mechanism at the moment, or cut out a small section of their release documentation, or decide not to test an underused area of the application. As they chip away at the overhead of deploying, the closer they’ll get to the benefits of regular user validation and risk mitigation outweighing the time and effort cost. Plus, once you start deploying often, everyone will start to focus on removing the remaining overhead.

We’ve been making a concerted effort to apply an agile software development approach at Redgate since mid-2008, but it still took us five years until we challenged our development teams to deploy more frequently. In our efforts to meet this challenge we’ve made some mistakes and we deployed some releases that we shouldn’t have. However, we made incremental improvements, we removed or worked around bottlenecks in our deployment process and we added mechanisms that allowed us to spot problem releases.

It is worth mentioning that the single most important improvement we made was giving every team the power to deploy their software at the press of a button. Redgate’s excellent DevOps team delivered a bespoke automated deployment tool that allowed us to do this. It involved significant investment on Redgate’s part but it was worth it, as it means that my team can deploy our products as often as they like. In effect, we are only limited by our ability to break work into discrete, valuable chunks and our users’ tolerance of software updates.

Software development teams should deploy as often as they can. Every month. Every sprint. Even every day, if your users would be happy with that. Reducing the gap between deployments reduces the amount of unproven changes that have not been validated by your users and lowers the risk of a single release having a plethora of pointy-fanged bugs ready to eat your time. Also, it means deployments will stop feeling like trips to sit in the dentist’s chair.

---

This is a guest post by Chris Smith, Head of Product Delivery at Redgate. Redgate helps you meet the needs of the business and IT, with solutions that accelerate software delivery and aid compliance with data protection regulations. [Learn more](https://www.red-gate.com/).
