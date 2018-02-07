---
title: "Discontinuing the free Octopus Community edition tier"
description: "Today we decided to discontinue the free Community edition of Octopus Deploy. This post explains why."
author: paul.stovell@octopus.com
visibility: public
published: 2018-02-07
tags:
 - Company
---

Since we started Octopus in 2012, we offered a free "Community" version of Octopus, and it has always been heavily used. In fact, about 80% of companies using Octopus today are using the free tier. Over the years we've tweaked it, and considered removing it many times. Today, I decided that we would go ahead and remove it. In this post, I'd like to explain the reasoning why. 

Before explaining why we've removed the free tier, it's worth asking: why do we charge for licenses at all? 

We think that in order to successfully help people automate their deployments, we need more than just software. We need well-written documentation. We need a good user experience. We need a website that explains the product. When new cloud technologies emerge or new frameworks are added, we need to add them. We need to support everyone who uses it, even if they get stuck using Octopus with other tools (build servers, cloud platforms, scripting languages) and the fault isn't with Octopus. Octopus could have been free and open source, and I'm sure it would attract a great deal of contibutors for code, but it would never have the level of polish, documentation and support it has today. 

Doing all of these things requires a team dedicated to working on Octopus and helping customers every day. My decision to remove the free tier comes from three main reasons:

## 1. Demands on support

As the number of people using Octopus grows, so does the demand on our support team. In 2012 Octopus was used by early adopter types, and we only did NuGet packages on Windows, so the support demand wasn't terribly high. Over time the number of features in Octopus, the number of deployment targets, and the number of application types we can handle has grown - and so have the support tickets. 

We deal with over 150 new support tickets every week, and send about 300 replies. We're continually improving the user experience, documentation and error messages to put downward pressure on those tickets, but we've never been able to reduce the number of tickets faster than the community has grown. 

We don't require you to provide license info when you contact support, and we respond to everyone the same, and I don't think it would match our culture to change either of those approaches. So when you consider that 80% of installs are on that free tier, it's a huge amount of work given away for free. 

## 2. Demands on the roadmap

Not unlike most software companies, Octopus makes a significant portion of our revenue from enterprise licenses - about 80% of our revenue comes from a small number of companies on licenses $5000 and above. And 0% of our revenue comes from the 80% of customers on the free Community tier. 

What this has meant is that we've continually put enterprise customers first when designing the roadmap or choosing how we spend our time. Without building enterprise features, we can't sustainably support all the companies using Octopus for free. Something about that doesn't feel right to me. I'd like to see a healthy balance, where smaller customers are paying a small amount for licenses ($750) but the sheer number of them means it's in our interest to build features and improve their user experience equally. 

## 3. Changing nature of deployments

Octopus prices based on the number of target machines you are deploying to. Many customers are moving towards the cloud, and in doing that are deploying to PaaS targets - which aren't counted towards licenses currently. If you deploy to 10,000 Azure web apps with Octopus, today it would still be free. And the support burden for a customer doing deployments at that scale is definitely non-zero. 

Even if we were to find ways to charge based on the number of PaaS targets you deploy to, there will always be edge cases. For instance, we're adding Kubernetes support to Octopus today, and it's quite possible that someone could be deploying to a massive hundred-node Kubernetes cluster with a free version of Octopus. That trend is only going to get worse.

## Exceptions

This change won't affect everyone (and probably won't affect you, yet). We have carved out a few exceptions:

### Open source and personal projects

If you are using Octopus at home for personal projects (rather than for work-related projects), we’re happy for you to use Octopus for free. If you want to use Octopus for an open source project that you are a contributor to, we’re happy for you to use Octopus for free. And if you are a startup or very small business, we’re happy to offer a 25% discount on the license. 

### What happens with existing users?

At the moment we're only applying this change going forward - if you started a trial prior to today, you'll be able to convert to the free tier at the end of the trial. If you're already using Octopus for free, this won't affect you. 

Yet.

I'll be doing some soul searching over the next few months and thinking about what the "right" way to handle existing customers already using Octopus for free should be. If you have thoughts on this, please let me know in the comments below. 

## Doesn't it lead to paying users later?

Yes, sometimes it does. I've seen customers switch to a paid version of Octopus after 12 months of using the free version. But it really doesn't happen as often as you would think. 

People on the free tier generally fall into a few camps:

 - Octopus adds amazing value, easily enough to justify $750, but they work at a company that makes buying software very difficult. In theory, the free Community edition should let them spend years seeing the value Octopus provides. In reality, after using Octopus for 12 months, they've never thought much about the value Octopus adds (since they never paid for it) and it tends to be a hard sale. 
 - Octopus adds amazing value, but we can't afford to pay. See some of the exceptions above. 
 - Octopus adds very little value, certainly not enough to justify $750 for. These customers simply don't have deployment problems complex enough for Octopus to add much value. They'll likely never buy. 

Of course, there's a strong chance that we're wrong - and removing this free Community tier is the worst strategic decision I'll ever make. 

## Summary

At this point you might say "bah, it's just greed - unbridled capitalism at its worst". I wish I could point to some noble, higher-purpose explanations as to why we're making this change. Unfortunately, it's really just economics: thousands of customers using Octopus for free results in a high support demand, and distorts our roadmap to focussing only on large enterprises to make up for it. My hope is that in making this change, we're able to make Octopus far more sustainable. 

The minimum entry price for Octopus is $750, which I know can seem like a lot. Certainly, you don't need Octopus to deploy your applications. You can use FTP, you can right-click publish in Visual Studio, you can follow a Word document, you can spend some time and script everything yourself. In that sense, Octopus competes with free, and $750 is a big asking price. 

On the other hand, if your deployments are a little more complicated, if you've ever seen engineers lost for weeks down the rabbit hole of scripting everything yourself, and if you've been bitten by incomplete Word documents, and you've tried Octopus, then you'll know the value Octopus provides. In that light I do believe Octopus is a worthwhile return on investment, and I hope this change won't lose you. 

Making a change like this is big, and I put off making this decision for 5 years because I honestly am not sure it's the right thing to do - but as I've watched our support demand grow, and gone through the yearly roadmap exercise, I'm more sure about it now than I have been in the past. I hope this helps to explain the reasoning behind the decision. 
