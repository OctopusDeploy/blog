---
title: "Support @ Octopus"
description: "We consider support a core part of Octopus, but what exactly happens when you need help?"
author: alex.rolley@octopus.com
visibility: private
published: 2018-02-09
tags:
 - Company
---
# Support @ Octopus

Support is part of the core of Octopus; not something that we consider an extra or a cost burden. So how, exactly, does that work in practice?

Easy, everyone (including our CEO) does support. We do have a support focused Operations team that handles most of the load; however every developer answers support queries. This goes both ways in that the Operations team can, and does, raise issues and check in code fixes for Octopus.

## What happens when I send an email/write a forum post?

As you probably already know, Octopus is an Australian company with all (well, all except one) of our staff based in Australia. An interesting side effect of this is that support requests from our overseas customers (US and UK especially) arrive overnight leaving us with a backlog of requests first thing in the morning.

We handle this is by performing a support triage call at 8:30 every morning, where we go through every ticket and find the best avenue for support (being either within the Operations team or escalating straight through to the developers). We keep this short (typically 30 minutes or less) and are looking for keywords rather than trying to diagnose every issue. To assist us we have an `Ownership` Trello board which has each team listed along with their support areas. 

**It's important to remember that speed, not 100% accuracy is the goal here.**

Assisting us in this process is a couple of homegrown tools, `Supportflow` and `Tendersmash`. `Tendersmash` is a frontend UI for our current support forum (`Tender`) that shows all of the newly created and unassigned forum posts. It allows us to quickly assign support requests to people and teams with a minimum of friction. `Supportflow` kicks in once a request has been assigned, creating a card in the appropriate Trello board inbox to alert a team to a support request. Some teams also have Trello > Slack integration enabled so that they are alerted in Slack when a ticket is assigned.

## Triage, got it. What next?

Typically most of our teams also have a morning stand-up, during which they quickly go over any support tickets that have been assigned directly to their team, again looking for keywords and anything that might assist in providing a quick and accurate resolution to any given issue.

Once our tickets have gone through this process they will all have an owner, and it is that person's responsibility to ensure the resolution of that issue. By resolution we don't mean `I fixed it for that customer`, we mean `I fixed it for this customer and all future customers`, either by updating code to resolve a bug, changing the UI to make an operation easier (or possible), or updating our documentation.

Because we’re focused on giving you the best experience we can, when responding to a request we try to think beyond the most obvious next step. We try, wherever possible, to anticipate other issues that could be connected to your case. If we can suggest possible avenues of investigation upfront it can often lead to a faster resolution just by indicating where we think the problem is (oh, they think it's a network problem, maybe I should check and see if there is a firewall here).

## Have we fixed it yet?

While we aim to resolve every ticket on the first response (you can't hit what you don't aim for, right?), we know this isn't always possible. Sometimes we need to implement a [code fix](https://github.com/OctopusDeploy/Issues), sometimes we don't have [enough information](https://octopus.com/docs/support/get-the-raw-output-from-a-task) to diagnose the problem, and sometimes it's just not that [easy to track the issue down](https://xkcd.com/1457/). What's important is that we always keep you informed as to what we are working on, and scale our response to match the impact to you (if you're reporting a production outage rest assured you will have a _lot_ of people working to get you online as soon as possible).

I hope this gives you an insight into the support process at Octopus, feel free to [reach out to us](https://octopus.com/support) at any point if you have any questions!


