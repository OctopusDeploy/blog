---
title: How deployments to physical branches would look without automation
description: We explore how difficult tenanted deployments to company branches would be if we didn't have automation.
author: andrew.corrigan@octopus.com
visibility: public
published: 2023-09-18-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - DevOps
  - Multi-Tenancy
---

Continuous Integration and Continuous Delivery's recommendation of deployment automation is *hugely* important for large organizations with complex software. That's especially true for multi-tenancy software delivered with [tenanted deployments](https://octopus.com/blog/what-are-tenanted-deployments).

But what if we didn't have deployment automation? How would tenanted deployments even work?

In this, the third of a trio of posts, we ask what tenanted deployments to physical branches look like using manual processes.

## A common tenanted deployment scenario for physical locations

You work for a large financial institution in the United Kingdom that has over 200 branches all over the country. Each branch has its own server and several computers that tellers use to help customers.

You must regularly deploy updates to local servers at all branches for the company's bespoke finance software.

## How the deployments would work without automation

There are a couple of options for delivering software like this without automation.

### Travel to each location

As with our [global deployments scenario](https://octopus.com/blog/global-deployments-without-automation), traveling to each branch to do updates is the easiest way to ensure things go smoothly.

Here, where our fictional company operates only in the UK, you wouldn't need to pay airfare, and your team could drive or take the train to branches instead. Similar organizations in larger countries like the US or Australia wouldn't have that luxury, however. 

Even if your team drives to your branches, travel would still cost time and money. Most organizations set up like this have the financial burden of fleet cars or fuel and travel reimbursement for staff. As for time, teams would waste days and months moving around the country to do updates.

For an added wrinkle, if your business went global, you'd end up with the *combined* pain points of 2 of our 3 scenarios. Software updates could become [Planes, Trains and Automobiles](https://www.imdb.com/title/tt0093748/?ref_=nv_sr_srsg_0_tt_8_nm_0_q_Planes%252C%2520t) style ordeals but without Steve Martin and John Candy's comedic timing.

### Have branch staff run the updates

Again, a bit like our [global deployments scenario](https://octopus.com/blog/global-deployments-without-automation), but here you'd have a branch staff member run the update for you.

It might sound strange to let an on-site, non-technical colleague run updates, but I worked at 2 organizations that did this. One of those only a few years ago. And yes, we'd mail out a disc rather than use better, more secure options. 

The problem is that things can and will go wrong no matter how simple you make the install or how you get files to branches. Updating software like this introduces a bunch of risky variables and room for error. Maybe there's a problem with the update itself. Maybe the instructions aren't clear for the non-technical. Maybe the last manual update missed some important prerequisites for new releases.

When there are problems, it's not a branch staff member's job to fix them. That means some of the time saved by not traveling may get spent troubleshooting over the phone or, much worse, moving the time cost to your business's service desk. 

Whichever way you cut it, if there's a problem, someone's time is getting wasted.

### Remote installation

In this scenario, you're deploying updates to your own organization's hardware, so remote access is the easiest and most sensible way to go. However, that might not be an option if you're delivering software to *another* organization's branches instead.

Remote updates would solve many deployment problems when dealing with brick-and-mortar locations, though. You can oversee the updates to ensure every step happens, that hardware's configured right, and that installs complete successfully. You can even run tests.

Though gaining access becomes quicker and easier, the process may not. Without deployment automation, you still have the problem of manual processes at scale.

We have over 200 branches to serve in the scenario we set at the top. That's a lot of tiring, repetitive work for a manual process, even if you divide the number of installs between a group of people.

### Okay, but what about scripts?

As with the other posts in this series, I think scripts count as deployment automation. We'll cover them anyway because there are potential and avoidable risks, even with scripting. 

That's largely because scripts can change depending on a given update and might not behave the same for every target.

Like the other manual deployment methods, you must think about scale when scripting, too. Experts could manage all branch deployments in one script, which is less work but can cause problems if any locations hit setbacks. Equally, a script per branch is a safer route and makes it easier to track failures, but that's more time-consuming - it's a lot of scripts to copy or write.

Writing a new script for every deployment means a huge chance of human error. You could make typos, get your syntax wrong, or forget to add a key step. Reusing older scripts may feel safer, but you risk carrying unknown errors forward - errors that could cause problems way down the line.

## The result 

Without deployment automation, whatever update strategy you pick will result in:

- Much slower software delivery - bad for bug or vulnerability fixing and delivering new features
- A higher risk of technical problems - bad for customer relations

And in this scenario, where updates can impact your frontline colleagues and your organization's customers, you can't really afford either.

Thankfully, deployment automation *does* exist. But not all deployment tools handle tenanted deployments as well as Octopus does.

Check out our [tenanted deployments use-case page](https://octopus.com/use-case/tenanted-deployments) to find out why.

You can also read our other 2 posts in this series:

- [How global tenanted deployments would look without automation](https://octopus.com/blog/global-deployments-without-automation)
- [How SaaS tenanted deployments would look without automation](https://octopus.com/blog/saas-deployments-without-automation)

Happy deployments!