---
title: Deployment Target UI Changes
description: Learn about some recent UI changes around deployment targets and why we made them.
author: shaun.marx@octopus.com
visibility: private
bannerImage: blogimage-ltsrelease.png
metaImage: blogimage-ltsrelease.png
tags:
- Deployment Targets
- Usability
---

We recently changed the user experience when adding a new deployment target and wanted to take the opportunity to cover the reasons for making the change and what we hope to gain from these changes. Suffice to say, we are excited about it and hope it makes adding a deployment target a more pleasant experience overall.

## The problem

Over the last couple of months we have held numerous usability studies and noticed customers getting stuck at certain points when using Octopus. These studies made it painfully clear that we didn't do enough to explain terminology or difficult concepts and we needed to do more to help. The deployment targets screen is one such screen, where we had numerous customers get stuck asking about what the differences are between polling and listening tentacles. It's even more confusing to new customers that don't know what a tentacle is.

It's a little known fact that radio buttons don't necessarily scale very well for a large number of choices. We have also been introducing new deployment targets which exacerbates the problem. It felt like the number of choice reached a tipping point and therefore required some love.

## The Approach

We could have converting the radio buttons to cards and leave it at that, however, that didn't feel like it solved the understanding problem and would have been rather overwhelming. We therefore decided to move the deployment target selection to a separate screen in order to make the selection more focused. This also provided us with an opportunity to categorize and narrow deployment targets in a way that more closely mirrors our documentation. This along with imagery and additional help text as part of the cards should make things a lot easier.

There were also a number of other small changes added in order to improve the navigation between the various deployment target screens and to make certain things slightly easier to find. This includes adding breadcrumbs where possible and featuring tentacle download buttons more prominently.

## Conclusion

Diagrams, descriptions and timely links rock. <-- this may need some work.

