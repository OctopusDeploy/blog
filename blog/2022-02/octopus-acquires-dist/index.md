---
title: Octopus Deploy acquires Dist
description: Octopus Deploy has acquired Dist, a cloud-based container registry and artifact repository.
author: paul.stovell@octopus.com
visibility: public
published: 2022-02-01
metaImage: blogimage-octopusdeployacquiresdist-2022.png
bannerImage: blogimage-octopusdeployacquiresdist-2022.png
bannerImageAlt: An Octopus-branded crane unloading containers from a ship named Dist.
isFeatured: false
tags: 
  - Company
---

Today I'm thrilled to announce that Octopus Deploy has acquired Dist, a cloud-based container registry and artifact repository service. 

## Why Dist is a great fit

Dist was created with the goal of providing highly reliable, fast, and easy-to-use artifact repositories for containers and Java artifacts. 

We met the Dist team in October last year and found that we had a lot in common. Dist was entirely bootstrapped by two Sydney-based founders, Yun Huang Yong and Stephen Haynes, who have strong technical backgrounds and are passionate about the space, and we were very impressed with what they had accomplished. 

Before an application can be deployed, it needs to be packaged, for example into a Zip file or Docker container image, which is generally called an “artifact”. Each source code change results in a new version of the artifact, and since you might need to re-deploy an old release, you need to keep many versions of different artifacts and store them efficiently, usually in software called an “artifact repository”. 

Since the early days, Octopus Deploy has included a built-in artifact repository. This could be used for pushing NuGet packages, Zip files, Java JAR files, and other artifacts for deployment. Customers always found this convenient - one less service to manage, and importantly, it integrated with our [retention policies](https://octopus.com/docs/administration/retention-policies). Today, while we support many types of artifacts, we don’t currently support container images, so customers need to use a third-party solution such as Azure Container Registry or AWS Elastic Container Registry, making retention policies more difficult to manage. 

## How Dist will help our customers

Our mission at Octopus is to accelerate repeatable, reliable, and traceable deployments across cloud and on-premises. As we build out Octopus Cloud’s functionality, we want to provide customers with a convenient way to use their container images, and potentially other artifacts. Low latency, secure access, integrated with retention policies in Octopus can help you do this without having to manage additional services. The Dist technology and the expertise of the Dist team will be central to what we do next.

## How Octopus is maturing

Over the last few years, Octopus has enjoyed rapid growth, you can read about how [Octopus grew in 2021](https://paulstovell.com/octopus-deploy-2021/). In the past, if we identified functionality that was missing in Octopus Deploy, our first instinct would have been to spin up a team of engineers to develop that functionality, but as we’re growing and maturing we realize, starting from scratch in-house won’t always be the best way to serve our customers. 

Together with the Dist team, we'll make it even easier for software teams to deliver more features and reduce downtime, and with less effort to manage. I'm delighted to welcome the Dist team to Octopus Deploy and can’t wait to see what we build together! 

### Fireside chat about Dist

Director of Product, Michael Richardson, led the Dist acquisition process. He recorded a chat with Senior Solutions Architect, Derek Campbell, about how Dist technology will improve deployments with Octopus.

<iframe width="560" height="315" src="https://www.youtube.com/embed/0CAFUTWW6-k" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### Container registry in Octopus

If you’re interested in the container registry in Octopus, you can [sign up to the feature in our Roadmap](https://octopus.com/company/roadmap#container-registry). This keeps you updated with our progress and ways you can help shape the feature.

Happy deployments!
