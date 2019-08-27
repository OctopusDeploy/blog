---
title: Redesigned deployment process editor, Tenant cloning and more - Octopus Deploy 2019.4
description: Octopus 2019.8 introduces our redesigned deployment process editor, Tenant cloning,
author: rob.pearson@octopus.com
visibility: public
published: 2019-08-28
metaImage: blogimage-shipping-2019-8.png
bannerImage: blogimage-shipping-2019-8.png
tags:
 - New Releases
---

![Octopus Deploy 2019.8 announcement fireworks](blogimage-shipping-2019-8.png)

We are shipping Octopus Deploy 2019.8 which is a roll-up of some small yet fantastic features. The most visible of the improvements is our new streamlined deployment process editor. The largest  with a focus on enhancing the feedback loop in your CI/CD pipeline.  If you have ever wanted better traceability of your requirements through build and deployment, this update is for you. 

This release includes some great updates: 


## Streamlined deployment process editor 

We're please to ship the first iteration of a more streamlined deployment process editor with side-by-side step editing and process filtering. 

## Tenant Cloning support



## New Variable Filter expressions

This release also includes new variable filter expressions. Whoa. That's a mouthful. Although this sounds confusing, it enables teams to do some really powerful transofrmations of their variables in scripts and blah.

## Configurable health checks 




## Other small enhancements that improve deployments

This release also incldues some other small improvements 

* Allow overriding namespace in Kubernetes steps
* Deploy a Release steps can now be used in rolling deployments
* Support providing certificates via portal by directly pasting as text

## Breaking Changes

`TODO!!!`

## Upgrading

As usual, please follow the [normal steps for upgrading Octopus Deploy](https://octopus.com/docs/administration/upgrading). Please see the [release notes](https://octopus.com/downloads/compare?to=2019.8.0) for further information.

* Self-hosted Octopus customers can start using these features today by installing [Octopus Server 2019.8](https://octopus.com/downloads). Note `2019.8` is a fast lane release without [long-term support](https://octopus.com/docs/administration/upgrading/long-term-support). This featureset will be included in the next [LTS](https://octopus.com/docs/administration/upgrading/long-term-support) release of Octopus at the end of Q3 2019.

* Octopus Cloud customers will start receiving the latest bits in about 2 weeks during their maintenance window.

## Wrap up

That's it for this month. Feel free to leave us a comment and let us know what you think! Happy deployments!
