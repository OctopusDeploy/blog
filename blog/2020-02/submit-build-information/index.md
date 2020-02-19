---
title: "Submitting build information using the API"
description: "Submitting build information using the API"
author: shawn.sesna@octopus.com
visibility: private
bannerImage: 
metaImage: 
published: 2021-02-05
tags:
---

Octopus Deploy has done a fantastic job of integrating with popular build servers such as Azure DevOps, Jenkins, and TeamCity.  The available plugins allow for packaging artifacts, pushing the artifacts to Octopus, creating releases, and even initiating deployments!  Along with these great features is the ability to include release notes and commit information with the build, referred to as `build information`.  While this is cool, what about the other build technologies where there aren't plugins available such as CircleCI (rumor has it that there is an Orb in development) or where there isn't even a build server such as using GitHub as a feed?  There's an API for that!  Since Octopus Deploy is written API-first, we are able to utilize the API to programatically submit build information!