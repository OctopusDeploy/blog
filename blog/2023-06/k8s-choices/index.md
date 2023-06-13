---
title: What you should consider before building CD for Kubernetes
description: There is more than one approach to deploy to Kubernetes; some decisions made in the beginning will be hard to change later. This post concerns a few things one should consider before configuring CD.
author: nikita.dergilev@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - DevOps
  - Kubernetes
---

Kubernetes is a powerful but complicated platform. One has to make multiple architectural decisions before moving to Kubernetes. Here are just a few of these: 
- choosing between self-hosted and managed,
- considering app architecture and which parts to run on Kubernetes (e.g. only stateless services or migrate stateful components as well),
- security (RBAC, network),
- how to organise the network layer.

In this blog post, I will focus on just one impactful decision — how to organise your CD pipeline and manage Kubernetes configurations. 

## Body

The body of the post is where you share your hypothesis, how-to, or story.

If there are any previous posts on the same topic, please link to them to help with our SEO efforts. For example:
Our post about [DORA metrics](https://octopus.com/blog/dora-metrics-devops-business-outcomes) discusses how agility-based metrics can help improve profitability, market share, and productivity. 

### Subheadings

Use three ### to include H3 headings.

Use **Bold** text for UI labels, use single back-tics for `parameters` and `filepaths`, and three back-tics for code blocks:

```
Write-Host "Hello, World!"
```

Use the following (minus the backtics) to include images:

```
![Alt text, a description of the image](/path/to/image.png "width=500")*Optional caption text*
```
If including images, please include alt text. Alt text is primarily used to describe images to people unable to see them, and can be 125 characters max including spaces. You can also include an image caption if the reader would benefit from additional information or context.

## Conclusion

Close off the post by restating the main points of the post, share any closing thoughts, and invite feedback.

## Learn more

- [link](https://www.example.com/resource)

## Register for the webinar: {webinar title here}

Short webinar description here, for example: A robust rollback strategy is key to any deployment strategy. In this webinar, we’ll cover best practices for IIS deployments, Tomcat, and full stack applications with a database. We’ll also discuss how to get the rollback strategy right for your situation. 

We're running 3 sessions of the webinar, from {webinar dates here, for example: 4 November to 5 November, 2021.}

<span><a class="btn btn-success" href="/events/rollback-strategies-with-octopus-deploy">Register now</a></span>

## Watch the webinar: {webinar title here}

<iframe width="560" height="315" src="https://www.youtube.com/embed/F_V7r80aDbo" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

We host webinars regularly. See the [webinars page](https://octopus.com/events) for details about upcoming events, and live stream recordings.

Happy deployments!
