---
title: Continuous Delivery of Octopus
description: Delivering Octopus Deploy faster and with higher quality
author: Matt Richardson
visibility: private
published: 2021-03-02
metaImage: to-be-added-by-marketing
bannerImage: to-be-added-by-marketing
tags:
  - Product
  - Engineering
---

## Nightly builds and continuous deployment

For a while now, we've been on a long-term missing to increase the flow of work and reduce time to feedback across all our teams. At the start of 2021, we were still facing some key constraints:

* Our deployment pipelines frequently suffered bit rot
* Our code changes often sat on branches that lived longer than we wanted

As of this week, we've just delivered some key changes to help us ship faster and better quality releases.

### ⏩ We are now practicing [Continuous Deployment](https://en.wikipedia.org/wiki/Continuous_deployment) of Octopus Server, Tentacle and Octopus CLI to internal customer environments, and [Continuous Delivery](https://en.wikipedia.org/wiki/Continuous_delivery) to external customer environments. What does this mean you ask?

Well, instead of us having to make a deliberate decision to deploy a release, we now automatically create a release from every commit to a releasable branch that passes all the tests. Successful builds roll out to our internal environments and then after a suitable "bake time", onto Octopus Cloud, and then to the website. This means we are constantly "drinking our own champagne" in the pursuit of delivering a quality product.

### 🌃 Nightly builds

For our older LTS releases, we were finding that "bit rot" was causing our pipelines to fail as we were only exercising them when we needed to ship something. Now, we are triggering a build every night so that we know that when we need it, it's definitely working.

### 🔢 major.minor.build versioning

Nightly builds did surface an interesting challenge for us though. With our prior numbering scheme, rebuilding every night would result in the same version number. Many downstream systems (eg nuget.org) dont like different packages with the same version number, so we had to come up with a new plan. We're now using a major.minor.build numbering strategy. This will mean that the build number will be much larger than we're all used to, but effectively, it's just a number. As this is now a build number, there will be gaps between version numbers. You'll almost always want to grab the latest build. 

As a side note, our journey down this path meant we diverged quite far from what [GitVersion](https://github.com/GitTools/GitVersion) is designed to do, so we ended up writing our own version calculator, [OctoVersion](https://github.com/OctopusDeploy/OctoVersion). This handles our multiple release streams much better, and as it's laser focused on our use case, it's much faster too.

### 📝 More accurate calculation of release notes from version `X` to version `Y` using the Git revision graph instead of GitHub milestones

Another interesting change in the mix here is around how we generate our release notes. We previously used GitHub milestones, and assigned them before we built a release. Now, as we're building a release for every commit, we are now using the Git revision graph to calculate what fixes went into what release. This means more accurate release notes.

### 🚷 Better handling of valid/invalid upgrade path

Now that we're using the Git revision graph to calculate these release notes, we also get some good knowledge about viable upgrade paths. Previously, it 2020.4.13 -> 2020.5.0 would appear to be a valid upgrade path, but would actually be going back in time (as 2020.5.0 was branched before 2020.4.13 was created). This occaisionally caused some bugs where the database had structural changes that were not expected. Now, we show a warning saying that this is not a viable upgrade path, meaning an entire class of bugs gets avoided.

## Conclusion

We're pretty excited about these changes - they'll help us focus less on the process and more on shipping good stuff. 
