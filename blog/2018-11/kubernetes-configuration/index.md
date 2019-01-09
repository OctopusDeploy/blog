---
title: Deploy to Oracle Database using Octopus Deploy and Redgate
description: Octopus Deploy supports many database tools.  Follow along as we get a CI/CD pipeline built to deploy a database change to an Oracle Database
visibility: private
published: 2019-10-16
metaImage: metaimage-redgate-database.png
bannerImage: blogimage-redgate-database.png
tags:
 - Database Deployments
---


* In typical container-based environments, we tend to think of the containers as being an

* When using containers the prevailing mindset in the community is that the container is an immutable deployable artifact. This is a perfect extension of the philosophy that Octopus follows with standard application packages; ["Build one deploy and many"](https://octopus.com/blog/build-your-binaries-once). One difficulty that 

* Modify container
* Environment variables
* Volume Mount (Config map)
    - Mount configuration as file
* External config source
