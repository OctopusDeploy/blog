---
title: What are slugs and why we chose them for OCL
description: TODO
author: eoin.motherway@octopus.com
visibility: public
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - Engineering
  - Configuration as Code
---

When we started developing the Configuration as Code feature, we made the decision to replace IDs with names in OCL.

This post explores the tradeoffs made by using names in OCL, what slugs are, and how they solved most of our problems with names.

## Traditional IDs

Octopus traditionally uses IDs to reference shared resources in things like deployment processes. The Octopus REST API returns JSON payloads containing IDs, and the web UI would do the heavy lifting, performing the necessary lookups to show user-friendly names.

```json
{
  "Steps": [
    {
      "Actions": [
        {
          "Environments": ["Environments-1"],
          "WorkerPoolId": "WorkerPools-2"
        }
      ]
    }
  ]
}
```

## Why Names in OCL?

IDs alone aren't very descriptive. The ID of an environment might look something like `Environments-24`. This isn't an issue when sending and receiving data over the REST API, as they're normally intended to be used by other programs. Unlike the JSON we use throughout the API, OCL is meant to be written and read by humans, so these IDs aren't very helpful.

If we use IDs in OCL, then the deployment process can be quite hard to understand at a glance.

```ocl
step "Run a Script" {
  action {
    environments = ["Environments-1"]
    worker_pool = "WorkerPools-2"
    ...
  }
}
```

It's not clear which environment is `Environments-1`, or which worker pool is `WorkerPools-2`. To find that out, you would need to double check using the web UI, or the API, which isn't a great user experience if you're reading or editing the OCL directly.

We initially solved this by replacing IDs with the resources name when converting projects to version controlled.

```ocl
step "Run a Script" {
  action {
    environments = ["Staging"]
    worker_pool = "Hosted Ubuntu"
    ...
  }
}
```

Looking at the above OCL, it's much easier to understand at a glance.

## The Tradeoffs

While using names can improve the readibility of the OCL, it presents a number of drawbacks.

### Names can be anything

Octopus is pretty relaxed about what kind of characters are allowed in names. This is great for names, but when we need to use these names to reference things from OCL, the names can become a bit awkward to work with.

```ocl
step "Run a Script" {
  action {
    environments = ["Staging (AU-EAST)"]
    worker_pool = "Alices Pool of Workers \"Pluto\""
    ...
  }
}
```

In these names, we've got some punctuation characters, mixed casing, escape characters, it's a bit of a mess, and could easily lead to mistakes.

This can also lead to confusion, as IDs are also technically valid as names. If you were to create a project called `Projects-5`, it would be hard to tell the difference between the projects name and ID out of context.

### Is it a name, or an ID?

Throughout the Octopus codebase, we generally assume that all references to shared resources are made using IDs. When we introduced names in OCL, we broke this assumption.
Now, ID fields could contain either a name, **or** an ID. This change broke a lot of existing code and introduces a number of bugs.

We found ourself regularly coming across edge-cases where we still assumed IDs were in use, but names were present instead. We even found places using both names **and** IDs at the same time, causing a lot of headaches.

### Breaking changes

Names as IDs also spread to our REST API, meaning API consumers now had to be aware of names and IDs in JSON too.

```json
{
  "EnvironmentId": "Staging (AU-EAST)",
  "WorkerPoolId": "Alices Pool of Workers \"Pluto\""
}
```



## Slugs to the rescue!

With all of the grief caused by names and IDs, we needed another solution to names.

### What are slugs?

Slugs are human-readable, URL-friendly, unique identifiers. They're often used in places where names wouldn't be a great fit, such as URLs, including the URL for this post!
For example, a name might be something like `Test Environment (AU-EAST)`, but if we slugify the name, we get `test-environment-au-east`.

### How we use slugs

We've added slugs to a number of different resources within Octopus, and we plan to add slugs to more documents in the future. This currently includes
- Accounts
- Environments
- Feeds
- Lifecycles
- Teams
- Worker Pools
- Deployment Targets
- Projects
- Channels
- Deployment Steps and Actions

Octopus will automatically generate slugs based on the name when creating new resources. These slugs can be changed later if desired.

When Octopus reads OCL from a git repository, it searches for slugs and converts them to the IDs we're all familiar with.
This removes the need to account for IDs or names, and allows most API calls to be re-used between database projects and version controlled projects.

Since slugs are human readable, they also improve the OCL experience.

```ocl
step "run-a-script" {
  name = "Run a Script"

  action {
    environments = ["staging-au-east"]
    worker_pool = "alices-pool-of-workers-pluto"
    ...
  }
}
```

It's much easier to understand which environments and worker pool this is referring to now, and we get the added bonus of these slugs being unique.

### The future of slugs

We're planning on introducing slugs into more and more resource types in the future.
We're also planning on using slugs more heavily in other areas of Octopus.

Happy deployments!
