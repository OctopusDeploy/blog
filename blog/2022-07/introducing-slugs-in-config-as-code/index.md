---
title: Introducing slugs in Config as Code
description: A brief overview of slugs, and how they're used in Config as Code.
author: eoin.motherway@octopus.com
visibility: public
published: 2022-07-25-1400
metaImage: blogimage-introducingslugs-cac-2022.png
bannerImage: blogimage-introducingslugs-cac-2022.png
bannerImageAlt: OCL text editor
isFeatured: false
tags: 
  - Engineering
  - Configuration as Code
---

When we started developing the Configuration as Code feature, we decided to replace IDs with names in Octopus Configuration Language (OCL). This approach came with several drawbacks, though, leading us to replace names with slugs in OCL.

This post explores the trade-offs by using names in OCL, what slugs are, and why we chose to implement them for Config as Code.

## IDs in OCL

Traditionally, Octopus uses unique IDs to reference shared resources from places such as deployment processes, runbooks, variables, etc.

On their own, IDs aren't very descriptive, they're simply made up of the resource type, and a unique number. For example, the ID of an environment might look something like `Environments-42`.

IDs in Octopus lack context. An ID alone isn't enough information to know what that ID refers to. This generally isn't an issue for things like APIs, as API responses are generally designed to be read by other programs.

OCL on the other hand is meant to be written and read by humans. Using IDs in OCL would make it hard to understand at a glance, as this example shows:

```ocl
step "Run a Script" {
  action {
    environments = ["Environments-42"]
    worker_pool = "WorkerPools-4"
    ...
  }
}
```

Out of context, it's not clear what this is doing. What environment is `Environments-42`? Which worker pool is `WorkerPools-4`? These IDs aren't exposed in many places, so trying to figure out what they refer to can be challenging.

This isn't a great experience if you're just trying to read the OCL.

## Names in OCL

Rather than using IDs in OCL, we decided to use the names of resources instead.

Names in Octopus are unique enough that we could confidently use them to identify resources. Names are also easily identifiable at a glance, making the OCL more readable and user-friendly.

```ocl
step "Run a Script" {
  action {
    environments = ["Staging"]
    worker_pool = "Hosted Ubuntu"
    ...
  }
}
```

Looking at the OCL above, it's much easier to understand what's going on compared to using IDs in OCL.

After we decided to use names in OCL, we also decided to use names *in place of* of IDs in both the API and in the UI for version-controlled projects.

We did this to improve the user experience when viewing version-controlled deployment processes from the UI. If there were any broken references in the OCL (like a typo in a name), we'd show a warning where the broken reference was found, and allow it to be fixed from the UI, or the API

While this worked initially, it presented some long-term drawbacks.

## The trade-offs

Throughout the Octopus codebase, we generally assume all references to shared resources are made using IDs. When we introduced names in OCL, we broke this assumption.

Now, ID properties could contain either a valid ID *or* the name of a resource that may or may not exist.

This resulted in several breaking changes both internally and externally, as the _"names as IDs"_ approach also affected our API, meaning API consumers (including ourselves) had to be aware of names and IDs in responses too.

Because IDs are also technically valid as names, it can be unclear whether a given value is an ID or a name, leaving it up to the user to guess or assume whether a particular value is an ID or a name.

Looking at the JSON below, there are a few issues:

- Property names are suffixed with `Id`, however, names are present instead. This is confusing and misleading.
- It's unclear what the value of the `EnvironmentId` property is referring to.
  - Is `Environment-1` the name or the ID?
  - What if I have an environment with the ID `Environment-1`, and another one with the name `Environment-1`, which one is this referring to?

```json
{
  "EnvironmentId": "Environment-1",
  "WorkerPoolId": "Hosted Ubuntu"
}
```

Using names as IDs also results in chatty API clients, as consumers need to make additional requests to the API to be able to fetch resources referenced by name.

The generic list-style endpoints (E.g: `/environments?partialName=foobar`) would need to be used to find all matching resources, then the results would need to be filtered manually by the consumer. These requests might not be cheap, especially if pagination is taking place and multiple requests need to be made for a single resource.

Another minor issue caused by names in OCL was the inability to rename resources without breaking references. This was to be expected but left room for improvement.

## Introducing slugs

With all of the grief caused by names as IDs, we considered using slugs.

Slugs are human-readable, URL-friendly, unique identifiers. They're often used in places where names wouldn't be a great fit, such as URLs (including the URL for this post).

Slugs are often automatically generated based on a name. For example, an environment named `Test Environment (AU-EAST)` would generate `test-environment-au-east`, removing any characters which aren't URL-friendly while preserving the readability.

## Slugs in Octopus

To keep the scope small, we started by adding slugs to anything which can be referenced from a deployment process. So far, this includes:

- Accounts
- Channels
- Deployment actions
- Deployment steps
- Deployment targets
- Environments
- Feeds
- Lifecycles
- Teams
- Worker Pools

We'll add slugs to more resource types as the need arises.

Projects already had the concept of slugs, which mostly lined up with our own goals for slugs. We managed to re-purpose most of the existing project slug logic and apply it to the aforementioned resource types with a few adjustments.

- Any newly created or existing resources will have their slugs automatically generated based on their name.
- Slugs can be modified independently of the name, allowing resources to be renamed without breaking any references. *This changes the existing behavior of project slugs.*
  - Modifying slugs is generally not advised, as it can potentially lead to broken external references which will need to be updated manually (for example, deployment processes defined in OCL).

## Slugs in OCL

After we added slugs to all the necessary resources, we started working on using slugs in our OCL.

This turned out to be relatively simple, as we could re-use our existing ID to name conversion code with some added logic for converting between IDs and slugs.

We also updated our OCL syntax for deployment steps and actions, so that slugs can be specified separately from names.

```ocl
step "run-a-script" {
  name = "Run a Script"

  action {
    environments = ["staging-au-east"]
    worker_pool = "hosted-ubuntu"
    ...
  }
}
```

It's much easier to understand which environments and Worker Pool this is referring to now, and we get the benefit of these slugs being unique, and re-usable between projects and spaces.

## Isolating the abstraction leak

At the time of writing, using slugs to reference shared resources only makes sense in OCL. Neither our front-end nor back-end is ready to use slugs for referencing these resources, and updating these to do so would be a non-trivial task.

Because slugs can be used as a contextually unique ID, we can map between traditional IDs and slugs when reading and writing OCL from Octopus.
This allows us to use slugs in OCL, while using IDs throughout the Octopus server, API, and front-end.

If Octopus encounters a slug that it can't map to an ID, a validation error is returned, instead of allowing the broken reference. We can maintain the validity of the data we're working with.

While this sacrifices the ability to view and edit broken references from the UI and API, we believe this is the right direction as it brings our API back into a consistent shape, and improves the stability and predictability of Octopus server when working with OCL.

## The future of slugs

With much of the groundwork for slugs now complete, we can start adding slugs to more resource types and begin using them more heavily throughout the Octopus codebase.

Many other teams outside of Config as Code have shown interest in using slugs throughout their domains, so stay tuned to see how we use slugs next.

Happy deployments!
