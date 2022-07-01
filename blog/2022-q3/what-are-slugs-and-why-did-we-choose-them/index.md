---
title: What are slugs and why we chose them for OCL
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

This post explores the tradeoffs made by using names in OCL, what slugs are, how we went about implementing them, and how they solved most of the problems we ran into with names.

## IDs in Octopus

Traditionally, Octopus uses unique IDs to reference shared resources from places such as deployment processes, runbooks, variables, and much more. These IDs are also used in the API routes, and web UI routes.
When a page is loaded in the web UI, it will get the ID from the route, and use that ID to send a request to the API. This allows us to show names in the UI.

## Config as Code

Once we began working on Config as Code, we realised that our IDs weren't going to work very well in OCL.

On their own, IDs aren't very descriptive. The ID of an environment might look something like `Environments-42`. This isn't an issue when using IDs in the API, as other applications are usually the ones reading them, so IDs make it much easier for them to perform any necessary requests.
OCL on the other hand is meant to be written and read by humans.

If we used IDs in OCL, then OCL can be quite hard to understand at a glance:

```ocl
step "Run a Script" {
  action {
    environments = ["Environments-42"]
    worker_pool = "WorkerPools-4"
    ...
  }
}
```

If you came across this in your git repository, it wouldn't be very clear what this is doing. What environment is `Environments-42`? Which worker pool is `WorkerPools-4`? To find that out, you would have to use the Octopus API, CLI, or web UI, which isn't a great experience if you're just trying to read the OCL.

## Names in OCL

When we realised we couldn't use IDs in OCL, we looked at our models for something that could be used to uniquely identify resources. We weren't spoiled for choice here, since all we had was ID, and name.

Names in Octopus are unique enough that we could confidently use them to identify resources, and they were used directly in the web UI, so they were easily identifyable at a glance.

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

Now that we had decided to use names in OCL, we had to figure out how we were going to convert IDs to names when writing OCL.

## Converting IDs to Names

We could've easily written a converter that explicitly replaces all IDs with names, for example:

```cs
public async Task ConvertIdsToNames(DeploymentAction action, CancellationToken cancellationToken)
{
  if (!string.IsNullOrEmpty(action.WorkerPoolId))
  {
    var workerPool = await workerPoolDocumentStore.Get(action.WorkerPoolId, cancellationToken);
    action.WorkerPoolId = workerPool.Name;
  }

  ...
}
```

This could've worked, but it wouldn't scale very well.
As we add support for more resource types in OCL, and update our existing models, we would need to keep this converter up-to-date, which could easily be forgotten.

To address this, we leveraged tiny types, and introduced the tiny type converter.

Throughout the Octopus codebase, we use tiny types (Sometimes called [value objects](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/implement-value-objects)) to represent IDs. This allows us to provide domain specific context to primitive types. 

```cs
public class DeploymentAction
{
  ...
  public WorkerPoolIdOrName WorkerPoolId { get; set; }
  public ReferenceCollection<EnvironmentIdOrName> Environments { get; } = new();
  public ReferenceCollection<EnvironmentIdOrName> Channels { get; } = new();
  ...
}
```

By utilizing tiny types, we can programitally identify which properties contain IDs. This allows us to use a bit of [reflection](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/reflection) to iterate over our models, identify IDs, perform the necessary lookups, and replace any IDs with names.

```cs
public class TinyTypeConverter
{
  public void Convert(object obj)
  {
    var lookups = BuildIdNameLookups();
    foreach (var tinyTypeReference in TinyTypeIterator.EnumerateTinyTypes(obj))
    {
      tinyTypeReference.Convert(lookups);
    }
  }
}
```

## Converting Names to IDs

TODO

## The Tradeoffs

While using names can improve the readibility of the OCL, it presented a number of drawbacks.

Throughout the Octopus codebase, we generally assume that all references to shared resources are made using IDs. When we introduced names in OCL, we broke this assumption.
Now, ID properties could contain either a name, **or** an ID. This, combined with the fact that IDs are also valid names made debugging a bit of a nightmare.
This also revealed a lot of edge cases and unexpected scenarios when Octopus tries to figure out if a value is a name or an ID.

The "names as IDs" approach also spread to our REST API, meaning API consumers now had to be aware of names and IDs in JSON too.

```json
{
  "EnvironmentId": "Staging",
  "WorkerPoolId": "Hosted Ubuntu"
}
```

## The new IDs

With all of the grief caused by names and IDs, we sought out alternatives.

Outside of the Config as Code team, other teams were also looking for alternatives to the traditional IDs.
<!-- Todo: Which teams, and use cases -->

We eventually decided to begin investigating fullt-qualified slugs.

## What are slugs?

Slugs are human-readable, URL-friendly, unique identifiers. They're often used in places where names wouldn't be a great fit, such as URLs (including the URL for this post).

For example, an environment named `Test Environment (AU-EAST)` would have the following slug: `test-environment-au-east`.
These slugs are also unique, and could be used as a surrogate ID.

## Fully-qualified slugs

Fully-qualified slugs are slugs which are unique throughout the entire Octopus system. This can be achieved by joining multiple slugs together.

For example, the fully-qualified slug for a channel might look something like `spaces/default/projects/first-project/channels/beta`. From this slug, we know what space and project the channel belongs to.

> Q: Why do I need to know about the parent space and project in a channel slug?

Slugs are unique within their owners scope. So, two projects can have the same slug, as long as they live in different spaces, the same goes for channels in different projects, etc.
Knowing the fully-qualified slug allows us to uniquely identify resources out-of-scope.

## Implementing slugs

When we began working on slugs, we chose to add slugs to the resources whose IDs could be converted to names, with plans to add slugs to remaining documents later on.

We initially started prototyping fully-qualified slugs on a bunch of these resources, but we eventually decided to de-scope fully-qualified slugs, and opt for more simpler and lightweight approach to slugs.
This was primarily due to time constraints, a large number of breaking changes, and we didn't need fully-qualified slugs for config as code anyway.
Instead, we would persue a more lightweight slug solution, while making room for fully-qualified slugs in the future.

Thankfully, there was a bit of ground-work already done for us, as projects already have a concept of slugs. Project slugs are automatically generated based on their names, and re-generated when the name changes. These slugs are used in the project URLs for the web UI.
We were able to re-use this existing project slug generation code, and combine it with a new concept called slug providers

### Introducing Slug Providers

Slug providers live in our [document store](https://octopus.com/blog/config-as-code-persistence-ignorance) layer as a decorator. Their job is to ensure that slugs have been set on any documents which need one, and don't already have one, much like how a database will automatically set the ID for any new rows.
Because slugs are a required field, this helps us immensely with backwards compatability, as our existing database calls don't need to be updated for slugs.

```cs
public class SlugProvider<T> : ISlugProvider<T> where T : class, IHaveSlug
{
  public async Task EnsureSlugIsSet(T document, CancellationToken cancellationToken)
  {
    // Don't want to overwrite any existing slug
    if (!string.IsNullOrEmpty(document.Slug))
      return;

    document.Slug = SlugGenerator.GenerateSlug(document.Name);
  }
}

public class SlugProviderDecorator<T> : IDocumentStore<T>
{
  readonly IDocumentStore<T> inner;
  readonly ISlugProvider<T> slugProvider;

  public async Task Add(T document, CancellationToken cancellationToken)
  {
    // Make sure the slug has been set before proceeding to the next IDocumentStore
    await slugProvider.EnsureSlugIsSet(document, cancellationToken);
    await inner.Add(T document, CancellationToken cancellationToken);
  }

  ...
}
```

> Q: What if two names are similar enough that they can generate the same slug?

This is entirely possible, and we've accounted for this in our slug providers.
For example, if we have two projects with very similar names (`My Awesome Project`, and `My AWESOME Project!!!`), they will generate the same slug (`my-awesome-project`).

Project slugs addressed this by appending a number to the end of the slug until it was unique.

```cs
public class SlugProvider<T> : ISlugProvider<T> where T : class, IHaveSlug
{
  readonly IDocumentStore<T> documentStore;

  public async Task EnsureSlugIsSet(T document, CancellationToken cancellationToken)
  {
    // Don't want to overwrite any existing slug
    if (!string.IsNullOrEmpty(document.Slug))
      return;

    var slug = SlugGenerator.GenerateSlug(document.Name);
    slug = await UniquifySlug(slug);

    document.Slug = slug;
  }

  public async Task UniquifySlug(string slug, CancellationToken cancellationToken)
  {
    var existingSlugs = (await documentStore.All()).Select(d => d.Slug);

    var i = 1;
    var uniqueSlug = slug;
    while (existingSlugs.Contains(uniqueSlug))
    {
      var uniqueSlug = $"{slug}-{i}";
      i++;
    } 

    return uniqueSlug;
  }
}
```

## Slugs in OCL

Once we added slugs to all the necessary resources, we started working on using slugs in our OCL. 

We've added slugs to a number of different resources within Octopus, and we plan to add slugs to more resources in the future. This currently includes:
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

It's much easier to understand which environments and worker pool this is referring to now, and we get the added bonus of these slugs being unique.

### The future of slugs

---

Happy deployments!
