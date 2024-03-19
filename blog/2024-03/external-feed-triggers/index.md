---
title: External Feed Triggers
description: 
author: mark.coafield@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 
isFeatured: false
tags: 
  - Product
  - Platform Engineering
  - Kubernetes
  - Continuous Integration
---

Octopus Deploy is adding a new feature to expand our existing GitOps capabilities, by adding external feed triggers to our existing ARC triggers, utalising the feeds, container images and Helm charts already referenced in your deployment process.

## GitOps approach to continous delivery

Using a GitOps approach to continuous delivery results in the desired state being ‘pulled’ from the source, and it’s not difficult to see why this is a good thing: application and infrastructure state may diverge from the desired state either intentionally or via drift, and state reconciliation needs to occur whenever this divergence occurs, not just when pushed from the source.

Traditionally, a continuous delivery workflow has followed a ‘one and done’ approach to deploying a given build: one build is pushed to a CD server, which deploys that build. The CD server’s role is then complete until the next build is pushed.  Any changes outside what is explicitly pushed to it aren’t known to the CD server..

Following a pull paradigm, as detailed in the [GitOps Working Group’s principles](https://github.com/open-gitops/documents/blob/main/PRINCIPLES.md#pulled-automatically), switches this around - software agents are able to access the desired state at any time.


### A pull paradigm for containerized applications

When working with containerized applications, there’s a few ways to approach this.  

As explained in [this CodeFresh article](https://codefresh.io/learn/gitops/gitops-workflow-vs-traditional-workflow-what-is-the-difference/), one approach is to configure a deployment automator to identify any changes to an image repository, which then updates a YAML file in the configuration repository.  This change is then identified by a GitOps agent which pulls that change and updates the cluster to match.

Another approach is used in our new external feed triggers feature in Octopus Deploy: 
Octopus monitors registries used in your deployment process, and as soon as it detects a change to selected images or charts, it creates a new release.  Your existing lifecycle will then progress that release through your environments and to tenants, just like your releases do currently.

The details of these container images and helm charts are already known within Octopus Deploy; this means we are able to use the registry locations, image names, chart names, and the required credentials to do this monitoring, without the need to add or maintain this information anywhere else.

### Using external feed triggers

As part of this work, we have improved and consolidated trigger type selection into a single pop-up:
```
![Improved trigger selection pop-up](add-new-trigger.png "width=500")*The new trigger selection pop-up*
```
After selecting "External feed", and the name and description for the the external feed trigger are entered, the next 

External feeds triggers support release creation in channels, using any 
```
![Channel selection for trigger](add-new-trigger-channel.png "width=500")
```

```
![Trigger sources selection](add-new-trigger-sources.png "width=500")
```


## Conclusion

Close off the post by restating the main points of the post, share any closing thoughts, and invite feedback.

## Learn more

- [link](https://www.example.com/resource)
