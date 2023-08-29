---
title: How global tenanted deployments would look without automation
description: We explore how difficult global tenanted deployments would be if we didn't have automation.
author: andrew.corrigan@octopus.com
visibility: public
published: 2023-09-06-1400
metaImage: img-mtcampaign-globaltenanteddeploumentswithoutautomation-2023.png
bannerImage: img-mtcampaign-globaltenanteddeploumentswithoutautomation-2023.png
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - DevOps
  - Multi-Tenancy
---

Continuous Integration and Continuous Delivery's recommendation of deployment automation is *hugely* important for organizations with complex software. That's especially true for multi-tenancy software delivered with [tenanted deployments](https://octopus.com/blog/what-are-tenanted-deployments).

But what if we didn't have deployment automation? How would tenanted deployments even work?

In this, the first of a trio of posts, I ask what global tenanted deployments look like with manual processes.

## A common global tenanted deployment scenario 

You work for a team that delivers software to a large international organization. The organization has headquarters in American, European, and Oceanic regions.

You're based in North America but must deliver regular updates to your customer's on-premises data centers in these 3 locations.

## How the deployments would work without automation

There are a couple of options for delivering software like this without automation.

### Travel to each location regularly

Hopefully you like flying, as you'd need to tour all 3 regions to help your customers keep their software updated. Traveling and installing the software yourself would be the safest bet to ensure updates happen correctly every time.

Jet lag and time away from home aside, you'd have other problems too. There could be language barriers as you arrange access to adjacent systems or find suitable times for downtime or cutover. You could struggle to communicate problems properly if your work has a hitch.

Although you're flying out to update the software personally, time zones could still prove a problem. You might need to coordinate certain tasks with your team back home. That means someone staying up late or getting up early to ensure you can do your job.

Worst of all, your customers could go weeks and months without updates. That's a lifetime in modern software, where you can't fix bugs or patch vulnerabilities.

### Trust someone else to do it

To avoid paying regular airfares, you could let someone local do updates for you. It could be your customer's tech support team or local third-party contractors (if your customers would allow that).

Many enterprise organizations still take this route, regardless of their deployment strategy. Particularly if they deal through resellers.

If you're not an enterprise company working with a well-trained reseller, however, there are still risks.

You need trust and confidence to let someone else deploy or install your software. Even then, problems could arise that would be easily solvable if you were there. It's not a good look if a third party gets something wrong or you don't clearly communicate your instructions.

You'd also need to consider how you safely get your software's files to those working on your behalf. That means you could waste time posting physical media or managing ways to transfer files securely.

If something goes wrong, problems with time zones and language barriers remain a factor. There could be a problem only your team could solve, and you might need to be up through the night to ensure everything runs smoothly.

The probability of issues increases with the number of regions, too. Let's remember you're serving 3 regions in this scenario. That means different groups performing updates in each location. You could end up with one successful update in one region but fail in another, and the failure point could be who's performing the update.

### Remote installation

You might be thinking, '*The internet has been around for a while! We can install our updates remotely!*' Well, that's true, you *could*, technically. Whether you *can* also depends on several factors.

Perhaps your customer's policies won't allow you to remote access or transfer files to their infrastructure.

Let's say you *can* transfer your software remotely, though. That does remove things like travel, a need to trust others, and generally makes things easier. It still leaves you with some problems though, including some we already mentioned.

Time zones still make manual tenanted deployments a problem. Your remote upgrades must align with your customer's needs, like when your software gets the least use, or the outage window they assign.

Testing the upgrade works also makes that worse, as someone on-site will want to test everything's working as it should. Someone's losing sleep somewhere, whether it's you or someone else, especially if something goes wrong and you need to roll back.

### Okay, but what about scripts?

I think scripts would count towards automation in this manual deployment scenario. But they only partially solve the problems, though, so let's unpack using scripts.

All a script could do for this otherwise manual process is automate the key steps of the deployment. It might transfer your files, install dependencies, and press the big red button. Some scripts might also do some basic testing for you.

However, there's a problem with scripts alone. You have to track how often they change. 

If you're writing a fresh script from scratch for each region, you need to make sure they follow the same processes and don't have any typos. Copying and editing the script you already used for one region? You need to make sure you double-check those regional infrastructure differences.

Using scripts this way also leaves you open to drift. What if you miss an important change from one location's script to the next? What if you forget to update your script template if something big changes? What if one has an error that's not critical right now but might be in 3 to 4 upgrades time? That can be hard to unpick when it happens.

## The result

Without deployment automation, whatever update strategy you pick will result in:

- Much slower software delivery - bad for bug or vulnerability fixing and delivering new features
- A higher risk of technical problems - bad for customer relations

And in this scenario, where your customer is a huge enterprise organization, you can't really afford either.

Thankfully, deployment automation *does* exist. But not all deployment tools handle tenanted deployments as well as Octopus does.

Read our [tenanted deployments use-case page](https://octopus.com/use-case/tenanted-deployments) to find out why.

Happy deployments!