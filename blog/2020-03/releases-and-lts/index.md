---
title: Octopus Releases and Long Term Support (LTS) 
description: We're iterating on the way we deliver releases of Octopus Deploy 
author: michael.richardson@octopus.com 
visibility: private
published: 2020-03-12
tags:
 - Product
---

We're iterating on the way we deliver releases of Octopus Deploy.  The highlights are: 

- We will ship 6 feature releases per year (`2020.1`, `2020.2`, ...) 
- Every feature release will receive critical patches for *6 months* 
- We will no longer explicitly mark releases as LTS (Long Term Support)
- Releases will be rolled out to Octopus Cloud instances before being made available to download for self-hosted instances  

We are confident that this will result in clearer messaging, even more stable releases, and a better overall experience for both our customers and us.

These changes take effect immediately. The 2020.1 release is currently being rolled-out to Octopus cloud instances, and will be made available for download very soon.
This release will receive patches (`2020.1.1`, `2020.1.2`, etc) for the next 6 months. The next feature release (`2020.2`) will begin roll-out to cloud instances in April, and will be available for download in early May. 

The points above are the key messages to take from this post. But for those interested, we'll also provide a little historical context and hopefully a window into our thinking.    

The way a software company makes new releases available is one of the most fundamental relationships with their customers. At Octopus, our delivery process has evolved, and while each decision was the rational one at the time, we were not satisfied with the current state. 

Originally Octopus had only a self-hosted product (no Cloud Octopus).  For a product which users download and install on their own infrastructure, their is a tension: _the releases you want to make the most noise about are also the least stable_.  The `.0` releases, which contain the new features, are the most exciting but also the most disruptive.  They have the highest installation rates, but are also the most likely to contain issues.  This is not an ideal combination, for either our users or us.  For many of our customers, stability is more important than the latest features, and when they asked "which is the most stable release we can upgrade to?", we couldn't always honestly answer with the latest.  So the [LTS program](https://octopus.com/blog/long-term-support) was born...      

By marking certain releases as LTS, we were attempting to give people the choice: do you want the very latest features, or the most stable release. In a sense this program was successful; it achieved exactly what it was intended to.  But it makes us sad when a large portion of our users don't have the newest features. We also don't believe it's the best experience for new users.  When a user comes to the Octopus downloads page for the first time, we are making them choose between stability and the latest features.  We want to give them both! 

In amongst this, we released a [cloud-hosted Octopus](https://octopus.com/docs/octopus-cloud) product.  This changes the game.   
Perhaps the biggest downside of shipping self-hosted software is that if a release contains an issue, there is no way to automatically upgrade everyone (many of our customers use Octopus in environments without internet connectivity). It is always painful for us to see users encountering issues for which we have already released fixes. When we host Octopus, if one user encounters an issue, we can rollout a resolution to all instances immediately, greatly reducing the impact. 
For this reason, it makes much more sense for us to roll-out new releases to our cloud users first.  This allows us to manage the rate of the roll-out (deploying to a small number of instances initially), and to stabilize the release quickly.  Once we are confident in the stability of the release, we will deploy to all cloud instances, and make it available for self-hosted customers to download. This also improves the experience for users of Octopus Cloud.  Today, we announce new features when they become available for download.  At this point these features are not available to cloud customers, and we can't even deterministically say when they will receive them.  From now on, whenever we announce a new feature it will be available to all Octopus customers immediately.   

If you have any questions or concerns about this, please reach out to us at support@octopus.com

Happy Deployments!
