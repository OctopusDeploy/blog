---
title: "Reflections on 2017"
description: "A recap of 2017, including progress on our roadmap and a general company update"
author: paul.stovell@octopus.com
visibility: private
metaImage: metaimage-shipping-4-2.png
bannerImage: blogimage-shipping-4-2.png
tags:
 - Company
---

In January 2017 I outlined our [product roadmap for 2017](https://octopus.com/blog/roadmap-2017). With 2017 firmly in the rear view mirror, I wanted to take some time to reflect upon our progress for the year. This post won't just be about the roadmap though - I also want to give you some insight into Octopus as a company and how we've grown over the year, and share some of the challenges we've had. 

## Kicking off

We start each year with a "kick off" event, and in 2017 it was held in our office in Brisbane. At the time we were 23 people (21 in Brisbane, Dalmiro in Argentina, and Matt Richardson in Melbourne). We use the Kick Off event to try to get everyone on the same page for the year. We started a few new things this year: 

 - A roadmap for the year (previously, we made it up as we went)
 - Some goals and KPI's for the year
 - We split the company into teams, giving each team an area of the product to own

## Growing the team

One decision we made midway through the year was to expand towards hiring people from other cities in Australia. Most people at Octopus work from home, but we have an office in Brisbane that is used one or two days a week. Many conversations take place in the office, and we were worried that remote folks would feel left out. We conducted an experiment where we locked the office for a few weeks, which went well. 

The result is that we were able to hire a good number of incredibly smart and professional people throughout 2017, many of which weren't based in Brisbane. Here's a photo from our 2018 Kick Off event with everyone together. It makes me so proud to work with such a fantastic group of folks. 

![Octopus team at Kick Off](reflections-2017-team-kickoff.jpg)

If you're curious, here's how the roles break down. 

![Growth and role breakdown](reflections-2017-people.png)

As you can see, Octopus is an incredibly engineering-heavy company - only two people out of 39 aren't working in areas of product development, design, operations or support. That's a deliberate choice; it's our belief that we can build and sell a world-class product by focusing on building a great product and supporting it well. Whether that's something we can maintain in 2018 isn't so clear. 

## Monthly feature releases

In April 2017 we introduced another change - we started shipping monthly feature releases. Previously, we'd ship new features and bug fixes whenever they were ready, with no real planning. From April, we changed how we plan releases:

- One monthly feature release (major/minor - e.g., 4.1). This contains new features and improvements. We make a bit of noise about these on the blog, newsletters, and so on. 
- Bug fixes and minor improvements (e.g., 4.1.1, 4.1.2). We release multiple of these through the month. 

For customers, this seems to have generally worked very well. You know to expect a release each month and can plan Octopus server upgrades around that. You know you can ignore most bug fix releases unless a particular bug affects you or you are on that major/minor release. For us, it's made it much easier to plan, document and test releases. 

## Roadmap progress

In the [2017 roadmap](https://octopus.com/blog/roadmap-2017) I called out a number of things we planned to do. As you might expect, our ideas around what is important change throughout the year, and unfortunately we didn't accomplish everything on the list. Below is a breakdown of some of the progress we made. 

###  UserVoice

I said we'd commit to trying to address all UserVoice suggestions with over 200 votes. Here's how the list stood at the time I wrote the post, and whether we achieved it or not. 

 - **714 votes**: [Improve the variables UI](https://octopusdeploy.uservoice.com/forums/170787-general/suggestions/7192251-improve-variables-ui)
   *Done. We shipped a big overhaul of this in 4.0 and improved it with 4.1.*
 - **566 votes**: [Composite step templates](https://octopusdeploy.uservoice.com/forums/170787-general/suggestions/12948603-composite-step-templates) 
   *Not done. We're still trying to work out the best way to go about this.* 
 - **562 votes**: [Update step templates across all projects](https://octopusdeploy.uservoice.com/forums/170787-general/suggestions/6072178-when-updating-a-step-template-update-across-all)
   *Done. This shipped in 3.12 in April.*
 - **460 votes**: [Dry run a deployment](https://octopusdeploy.uservoice.com/forums/170787-general/suggestions/6169634--dry-run-deployment)
   *Not done. Dry run a whole deployment isn't practical and doesn't really seem to be what this is about. There are some underlying stories/goals we need to uncover from this. I suspect everyone who voted on this has a different idea of what they actually need.* 
 - **459 votes**: [Support Azure Service Fabric](https://octopusdeploy.uservoice.com/forums/170787-general/suggestions/7790970-support-for-azure-service-fabric)
   *Done. We shipped this in 3.13 in May.* 
 - **345 votes**: [Environment and machine conditions should support both "and" and "or" conditions](https://octopusdeploy.uservoice.com/forums/170787-general/suggestions/5731857-environment-and-machine-conditions-should-be-inclu)
   *Done. This shipped in 3.7.* 
 - **277 votes**: [Permission attributes for variable sets](https://octopusdeploy.uservoice.com/forums/170787-general/suggestions/6986441-permission-attributes-for-variable-sets-library-v)
   *Not done. We're planning a permissions overhaul as well as a new feature called Spaces which limits how widely variable sets can be shared - I think these might solve most of these isssues.* 
 - **272 votes**: [Cloning of steps](https://octopusdeploy.uservoice.com/forums/170787-general/suggestions/6470009-cloning-of-steps)
   *Done. This shipped in March.* 
 - **242 votes**: [Allow step run condition to be based on a variable](https://octopusdeploy.uservoice.com/forums/170787-general/suggestions/6594872-allow-the-run-condition-of-a-step-to-be-based-on-a)
   *Done. This also shipped in 3.7.* 
 - **235 votes**: [Version control of Octopus configuration](https://octopusdeploy.uservoice.com/forums/170787-general/suggestions/15698781-version-control-configuration)
   *Not done. We're still debating the best way to go about this. Expect an RFC on this this year.* 
 - **213 votes**: [Output variables for offline drops](https://octopusdeploy.uservoice.com/forums/170787-general/suggestions/9196032-output-variables-for-offline-drops)
   *Not done. Might wait until [Remote Release Promotions](https://octopus.com/release-promotions) to resolve.*
 - **210 votes**: [Project dependencies](https://octopusdeploy.uservoice.com/forums/170787-general/suggestions/9811932-allow-project-dependencies-so-deploying-one-proj)
   *Not done. We'll make progress on this in 2018* 
 - **200 votes**: [Allow polling Tentacles to communicate on port 443](https://octopusdeploy.uservoice.com/forums/170787-general/suggestions/6703593-allow-polling-tentacles-to-contact-octopus-on-443)
   *Done. We shipped this in April.* 


Overall, that's 7 out of 13 done. Of the items that aren't done, they're typically either not entirely clear, or something that we think will be obsoleted by another bigger change in the future. 

### Octopus Ops

In the 2017 roadmap, I said we'd build first-class operations functionality into Octopus. 

> The Octopus dashboard shows you whether your last production deployment was successful. What if it also showed you whether what you deployed is still running? ... It wouldn't just be limited to monitoring status though: you could also start/stop these services. Did the Windows Service that you deployed yesterday to 30 machines suddenly crash on 7 of them? No problem, just click the button, choose the 7 you want to restart, and hit the execute button. Sure beats using remote desktop! Plus, there would be a nice audit trail.

Unfortunately, we didn't make any progress on this in 2017. When we split up the teams for 2017, this one fell in the middle, and no team really owned it nor did we have anyone to champion it. 

In 2018 we'll be focusing heavily on this, but that's a subject for another post. 

###Octopus Slack App 

In the 2017 roadmap we planned to build a first-class "ChatOps" experience for Octopus with Slack. Bots were all the rage in 2017, and we have one that we use internally for our own deployments with Octopus. Unfortunately again, there was no owner for this, so it fell through the cracks. 

### New Step Builder and a lot more IIS options

In the 2017 roadmap we planned to make some changes to how we present deployment steps:

> For deployment steps that configure things like Windows Services, there's only a handful of options anyway. But some steps, like those that configure IIS, have potentially hundreds of different settings you might want to set. For now, this has meant that we only expose the most common settings, and you have to write PowerShell to do the rest.

I think we mostly achieved this. We overhauled the UI in 4.0 and collapsed settings - this allows us to provide a lot more options for steps. We did increase the number of options for IIS throughout the year, including supporting certificates and a number of other settings, but you'll still need PowerShell for more advanced settings. 

### Lower the learning curve

This is something we made solid progress on, partly with a new onboarding guide that we added early in the year, and then again with the UI overhaul for 4.0. 

![UI overhaul and onboarding guide](reflections-2017-ux.png)

Existing customers probably didn't benefit from this directly, but it helped a lot of larger customers indirectly through making it easier for new teams in those organizations to get started. 

### Mono-free Linux (SSH) deployments

We made good progress here too. In August, we added options to SSH targets to not require Mono. So long as we know what platform you are running, we push a .NET Core version of Calamari, our deployment runner, to the machine and run the deployment without relying on Mono. Our goal will be to obsolete the Mono approach in favor of the .NET Core model. 

### Octopus Release Promotions

Unfortunately, aside from planning, we haven't made any progress on this so far. 

### "Swaggerize" our API

Swagger was added to Octopus in September. Just go to 

![Swagger support in Octopus](reflections-2017-swagger.png)

### Tighter AWS integration

We started this very late in 2017, and expect to ship some initial support quite soon. Watch this space!

### PaaS Octopus

This is a big project, and one that we found ourselves putting off, waiting for other things, and essentially talking ourselves out of starting for most of 2017. I convinced myself that we couldn't start this project until that project was done, or some other project, or some big architecture change, and so on. Essentially we had to eat an elephant, but were procrastinating about taking the first bite.  

A few months ago we realized just how much our procrastination on this was sending some prospective customers towards VSTS (more on that in a moment). Instead of over-thinking it and re-architecting Octopus in the process, we've focused on building something that will be as close to on-premises Octopus as possible, at a reasonable price, and that we can host reliably. Once that's shipped we can find ways to cloud-optimize it later. 

## Non-roadmap progress

We also made progress in a number of areas not directly related to our roadmap. 

###Frontend refactor - Angular to React

Octopus 4.0 was a complete UI overhaul, as we converted it from an aging AngularJS (1.0) UI to React. We started it in earnest with a smaller team in around April/May, and didn't ship it until early November - and by then half the developers at the company were involved in it. 

This is a project that was governed by Hofstader's law:

> Hofstadter's Law: It always takes longer than you expect, even when you take into account Hofstadter's Law.

One of the things I enjoy about running a software company is getting to "put my money where my mouth is". As a consultant, I'd often suggest it was time to rewrite this or that. It's hard to point to a particular "business reason" for needing to commit half of our development time for about half a year, or to justify the decision, beyond simply "we think that it will make the product better over the long-term". 

At Octopus, we've done this several times in the past:

- Octopus 1.0: Switched from SQL Server (EF) to RavenDB. 
- Octopus 2.0: Rewrote the UI (ASP.NET MVC to AngularJS 1.0), rewrote the Server/Tentacle communication stack
- Octopus 3.0: From RavenDB back to SQL Server, rewrote the communication stack again

Each of those turned out to be good investments that improved the product dramatically, even though 6 months in you start to worry that it's a gigantic waste of time, money and has a very high opportunity cost. Fingers crossed it will work out for us again! 

### Java

At the start of 2017 I hadn't planned for Octopus to do much in the way of Java (or non-.NET support), but this changed about 6 months in. In 2017, we:

- Built a plugin for Atlassian Bamboo
- Added a number of Java deployment steps to Octopus
- (4.1) Added Maven feeds and certificate support for Java deployments

## Business progress

Stepping away from the product for a bit, I want to reflect on the progress Octopus Deploy made as a business in 2017, and some of the challenges we're facing. If you mostly follow this blog for technical/product content, this might not be so interesting, but it might give you some context that will help you to understand our decisions to date. 

Octopus is privately owned, profitable, and has grown dramatically since I started it in 2012. We've grown from being a simple deployment tool developed in my spare time, to a company of 39 people. I don't think it's misleading to say that Octopus is the most popular deployment automation tool for .NET developers - we know that about 20,000 companies have an Octopus server installation online today, and those Octopus servers do millions of deployments a year to hundreds of thousands of deployment targets. 

We celebrated a number of successes this year. Octopus was the #3 fastest growing company in Australia over the past 3 years, according to [BRW Fast 100](http://www.afr.com/leadership/afr-lists/fast-100/financial-review-fast-100-2017-the-full-list-20171103-gzeg47). It was brilliant to see Octopus in print in a newspaper - we posted copies to relatives, most of whom still aren't exactly sure what Octopus is. 

Our challenges this year fell into two main categories: internal and external. 

As far as internal challenges, these mostly related to how we organized & planned as we grew. As the company grew, so did our ambitions. We had four very big projects in mind this year:

- Some big changes around [how Octopus scales out across large enterprises](https://octopus.com/blog/octopuses)
- Cloud-hosted Octopus
- Remote release promotions
- UI refactor

Most of these were so big, that we convinced themselves they had dependencies on each other. We shouldn't start Cloud-hosted Octopus until we make the big architecture changes. We shouldn't start Remote release promotions until we finish the UI refactor. And the architecture changes should wait for the UI refactor too. 

Something I've learned is that we at Octopus are at our best when we focus on smaller projects that improve the product bit by bit in an agile, iterative way. That's how we did the onboarding work, that's how we added Java support. When it comes to bigger projects like this, we tended to drift and lose our way, and as a result wasted a lot of time. In previous years we might not have even attempted such projects; in 2017 we were confident we could, but it seems like we're just not quite there yet. 

Our biggest external challenge in 2017, I'd say, came from our relationship with Microsoft. 

### Competition with Microsoft

If you build software in the Microsoft ecosystem, Microsoft are clearly the 800-lb. gorilla. For Octopus it's a complex relationship:

- We integrate heavily with Microsoft services, like Azure
- Our customers today are primarily in the Microsoft ecosystem
- Our product is built on the Microsoft stack
- Microsoft are also our biggest competition

The last point really hit home this year, and has defined most of our strategy over the last 6 months. 

Octopus helps customers to automate their deployments. A few customers use Octopus with Jenkins. Many use it with TeamCity. The majority - I'd guess about 60% - use it with Team Foundation Server. In 2013 Microsoft acquired a release management product called InRelease from InCycle software, and bundled it with TFS. It didn't gain much traction - it was overly complicated, had a big workflow editor, and didn't actually do a lot to help people actually deploy things. 

Around 2015, they began to rewrite the build pipeline in TFS/VSTS, and eventually rewrote much of both the build and release aspects - I doubt there's much of the original InRelease code left behind. 

It's not conventional wisdom as a CEO to mention a competitor, but VSTS today has a release management story that's gaining ground on Octopus. I think that for more complex deployments, Octopus still has a strong lead, but for simpler cloud-focused deployments I'm not surprised to VSTS RM gaining traction. 

Here's a summary of where we find ourselves:

- We're competing with Microsoft in an ecosystem that largely prefers Microsoft tooling
- VSTS RM is essentially free. Technically they charge for concurrent deployments; in reality even for very large enterprises, it's essentially free or in the very low digits. They know this. 
- Visual Studio 2017 introduced a right-click, "Configure Continuous Delivery..." option in the solution explorer that sets your project up with VSTS build and release management. Octopus isn't an option. 
- VSTS RM was in the keynote at Build this year, and they're doing a lot of work to promote it.

As an ISV, when you partner with Microsoft, there are a few upsides - you can get to the product teams a little more easily, as an example. And teams that have very little traction of their own - like an esoteric new Azure service that isn't getting a lot of usage - will be eager to reach out and help you build integrations. Some teams at Microsoft were kind enough to invite us to exhibit at Build, which was fantastic.  That partnership wasn't enough to insulate us when Octopus came into the VSTS crosshairs though. 

For the record, I think healthy competition is great. VSTS have a roadmap that brings it into closer competition to Octopus, and that's fine. Bring it on!

We've seen our new customer growth stall heavily over the last 6 months, and I think a big part of that is the competition with VSTS. And I'm reminded that the product is free, integrated with Visual Studio, and keynoting conferences with it, and I realize that we're in trouble. 

That might help to explain some of our direction in 2017/2018:

- We'll keep competing head-on with VSTS, starting with our own cloud-hosted service. I think my team is just as clever and technically brilliant as the teams at Microsoft, and I think we can beat them head on. 
- You'll see us diversifying outside of the Microsoft ecosystem quite a bit. Particularly, by supporting Java and other platforms, and integrating with AWS. It's simply too dangerous to be tied to the Microsoft stack. 
- We'll focus on our enterprise offering. For large enterprises and complex deployments, Octopus still has a very strong lead over VSTS, and we'll need to extend that lead in order to find ways to grow. 
- We'll continue to work with technical folks at Microsoft when it comes to adding new features that our users are asking for. But we're no longer partnered with Microsoft in a marketing/ecosystem sense, and we won't be going out of our way to add support for Microsoft solutions that our customers aren't asking for. 

I love what Microsoft are doing with .NET Core and the general openness Microsoft have been improving over the last few years (now that Azure makes it in their interests to play nicely with others). I have tremendous respect for many of the people I know at Microsoft and really want to do the right thing by customers. But as an ISV, I'm pretty disappointed with Microsoft as a partner. 

## Wrap up

In 2017, Octopus grew dramatically. We improved and matured how we work internally, and we made pretty good progress along our roadmap. We spent a lot of time rewriting the UI, something that slowed us down in the second half of the year, and we saw increased competition from Microsoft which had a real bottom-line impact. 

If you're interested in more behind-the-scenes stuff at Octopus, especially technical content, subscribe to our [YouTube channel](http://www.youtube.com/c/Octopusdeploy). Every week we post videos from our internal all-hands meetings where we demo what we've been working on. 

In my next post, I'll be sharing our roadmap for 2018. Happy deployments!