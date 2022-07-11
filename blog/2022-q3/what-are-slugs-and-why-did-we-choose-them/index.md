---
title: Introducing slugs in Config as Code
description: TODO
author: eoin.motherway@octopus.com
visibility: public
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 
isFeatured: false
tags: 
  - Engineering
  - Configuration as Code
---

When we started developing the Configuration as Code feature, we made the decision to replace IDs with names in OCL.

This post explores the tradeoffs made by using names in OCL, what slugs are, and why we implemented them for config as code.

## IDs in OCL

Traditionally, Octopus uses unique IDs to reference shared resources from places such as deployment processes, runbooks, variables, and many more. These IDs are also used in the API, and web UI routes.
On their own, IDs aren't very descriptive, they are simply made up of the resource type, and a unique number. For example, the ID of an environment might look something like `Environments-42`.
This isn't an issue when using IDs in the API, as other applications are usually the ones reading them, so IDs make it much easier for them to perform any necessary requests.

OCL on the other hand is meant to be written and read by humans. If we used IDs in OCL, then it can be quite hard to understand at a glance:

```ocl
step "Run a Script" {
  action {
    environments = ["Environments-42"]
    worker_pool = "WorkerPools-4"
    ...
  }
}
```

Out of context, it's not what this is doing. What environment is `Environments-42`? Which worker pool is `WorkerPools-4`? To find that out, you would need to use the API, CLI, or web UI to figure out which ID maps to which resource.
Thich isn't a great experience if you're just trying to read the OCL.

## Names in OCL

Rather than using IDs in OCL, we decided to use the names of resources instead.
Names in Octopus are unique enough that we could confidently use them to identify resources. They were also easily identifyable at a glance, making the OCL more readable and user-friendly.

```ocl
step "Run a Script" {
  action {
    environments = ["Staging"]
    worker_pool = "Hosted Ubuntu"
    ...
  }
}
```

Looking at the above OCL, it's much easier to understand what's going on comparaed to using IDs.

## The Tradeoffs

Once we had decided to use names in OCL, we also chose to use names **instead** of IDs for version controlled projects not just in the OCL, but also in the API and the UI in an attempt to improve user experience.
This presented a number of drawbacks.

Throughout the Octopus codebase, we generally assume that all references to shared resources are made using IDs. When we introduced names in OCL, we broke this assumption.
Now, ID properties could contain either a name, **or** an ID. This resulted in a number of breaking changes both internally and externally, as the _"names as IDs"_ approach also affected our API, meaning API consumers now had to be aware of names and IDs in responses too.

Because IDs are also technically valid as names, it can be unclear whether a given value is an ID or a name, leaving it up to the consumer (including ourselves) to guess or assume whether a particular value is an ID or a name.

Looking at the below JSON, there are a number of issues:
- Property names are suffixed with `Id`, since that what they traditionally are. However, names are present instead.
- It's unclear what the value of the `EnvironmentId` property is referring to.
  - Is `Environment-1` the name, or the ID?
  - Did something go wrong during the conversion, and this ID didn't get converted properly?

```json
{
  "EnvironmentId": "Environment-1",
  "WorkerPoolId": "Hosted Ubuntu"
}
```

Names as IDs also resulted in chatty API clients, as consumers need to make additional requests to the API to be able to fetch resources referenced by name.
The generic list-style endpoints (E.g: `/environments?partialName=foobar`) would need to be used to find all matching resources, then the results would need to be filtered manually by the consumer.
These requests might not be cheap, especially if pagination is taking place and multiple requests need to be made for a single resource.

One more minor issue caused by names in OCL was the inability to rename resources without breaking references. This was to be expected, but was certainly had room for improvement.

## Introducing Slugs

With all of the grief caused by names as IDs, we began looking into a new form of identifier.

We wanted something that was unique, human readable, resistant to renaming, and could be re-used between projects, spaces, etc.

## What are slugs?

Slugs are human-readable, URL-friendly, unique identifiers. They're often used in places where names wouldn't be a great fit, such as URLs (including the URL for this post).

For example, an environment named `Test Environment (AU-EAST)` would have the following slug: `test-environment-au-east`.
These slugs are also unique, and could be used as a surrogate ID.

## Slugs in Octopus

We started out by adding slugs to anything which can be referenced from a deployment process. As the need increases, we can add slugs to more resource types.

Thankfully, we had some prior art with slugs as projects already have a concept of slugs.
Project slugs are automatically generated based on their names, and re-generated when the name changes. These slugs are used in the project URLs for the web UI.
We decided to re-purpose most of the logic from project slugs, and bring them to the aformentioned resource types.

Any newly created or existing resources will have their slugs automatically generated based on their name. The slugs can be changed if desired, but is not advised unless necessary, as it will require any external references to it to be updated manually (E.g: deployment processes defined in OCL).
Slugs can be modified independantly of the name, meaning resources can be renamed without breaking any existing references. This changes the existing behaviour of project slugs.

## Slugs in OCL

Once we added slugs to all the necessary resources, we started working on using slugs in our OCL.
This turned out to be relatively simple, as we could re-use our existing ID to name conversion code with some added logic for converting between IDs and slugs.

We also updated our OCL syntax for deployment steps and actions, so that slugs can be specified separately from names.
Since slugs are human readable, they also improve the OCL experience compared to using IDs.

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

It's much easier to understand which environments and worker pool this is referring to now, and we get the added bonus of these slugs being unique, and re-usable between projects and spaces.

When Octopus reads the OCL, it will try to convert any slugs it finds back to IDs. If it fails to find a resource for a particular slug, it will throw an error, keeping the data in a valid state.
This means we no longer allow OCL to be read when it contains broken references, however, we think this is the right thing to do longer term as it allows us to maintain the integrity of the data we're working with.
It also means we don't need to guess or assume whether or not a value is an ID, a slug, or a name. It's either an ID, or it's invalid.

## The future of slugs

With much of the groundwork for slugs now completed, we can start adding slugs to more and more resource types, and begin using them more heavily throughout the Octopus codebase.
A number of other teams outside of Config as Code have shown interest in using slugs throughout their domains, so stay tuned to see how we use slugs next!

Happy deployments!
