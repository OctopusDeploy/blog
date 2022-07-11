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

This post explores the tradeoffs made by using names in OCL, what slugs are, and how we implemented them for config as code, and future features.

## IDs in Octopus

Traditionally, Octopus uses unique IDs to reference shared resources from places such as deployment processes, runbooks, variables, and many more. These IDs are also used in the API, and web UI routes.

## Config as Code

When we began working on Config as Code, we realised that our IDs weren't going to work very well in OCL.

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

If you came across this in your git repository, it wouldn't be very clear what this is doing. What environment is `Environments-42`? Which worker pool is `WorkerPools-4`? To find that out, you would need to use the API, CLI, or web UI to figure out which ID maps to which resource, which isn't a great experience if you're trying to read the OCL.

## Names in OCL

When it came to looking for an alternative for IDs, we weren't spoiled for choice. The only unique identifiers we could use were IDs, and names.

Names in Octopus are unique enough that we could confidently use them to identify resources. Since they were used directly in the web UI, they were also easily identifyable at a glance.

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

## Converting IDs to Names

Now that we had decided to use names in OCL, we had to figure out how we were going to convert IDs to names when writing OCL. We could've easily written a converter that explicitly replaces all IDs with names, for example:

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
As we add support for more resource types in OCL, and update our existing models, we would need to keep this converter up-to-date, which could easily be forgotten, and it's implementation could easily drift from the model definition.

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

By utilizing tiny types, we can programitally identify which properties are IDs. This allows us to use a bit of [reflection](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/reflection) to iterate over our models, identify IDs, perform the necessary lookups, and replace any IDs with names.

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

## The Tradeoffs

While using names can improve the readibility of the OCL, it presented a number of drawbacks.

The decision to use names in OCL was made before [document stores](https://octopus.com/blog/config-as-code-persistence-ignorance) were implemented into the Octopus codebase. We also lacked a solid caching layer between Octopus Server and the database.
Because of this, we had concerns about performance when converting between IDs and names every time we read from and write to OCL. We ultimately decided to convert IDs to names when conveting project to version controlled, then using names from then on.

This decision had a couple of side effeects. 
Throughout the Octopus codebase, we generally assume that all references to shared resources are made using IDs. When we introduced names in OCL, we broke this assumption.
Now, ID properties could contain either a name, **or** an ID. This caused a number of breaking changes, which combined with the fact that IDs are also valid names made debugging a bit of a nightmare.
This also revealed a lot of edge cases and unexpected scenarios when Octopus tries to figure out if a value is a name or an ID.

The _"names as IDs"_ approach also spread to our REST API, meaning API consumers now had to be aware of names and IDs in JSON too.

```json
{
  "EnvironmentId": "Staging",
  "WorkerPoolId": "Hosted Ubuntu"
}
```

The final nail in the coffin for _"names as IDs"_ was chatty API clients. Our API consumers would need to make additional requests to the Octopus API to be able to fetch resources referenced by name.
Because our API doesn't support lookups by name, consumers would need to use our generic list-style endpoints (E.g: `/projects`) to find all resources and manually search through their names.
These requests might not be cheap, especially if pagination is taking place and multiple requests need to be made for a single resource.

## Introducing Slugs

With all of the grief caused by names as IDs, a number of teams within R&D sought out alternatives.

We quickly came to a consensus and began investigating fullt-qualified slugs.

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

When we began working on slugs, we chose to add slugs to a handful of resources, with plans to add slugs to remaining documents later on, keeping the initial scope narrow, and allowing for room to grow in the future.

We initially started prototyping fully-qualified slugs on these resources, but we eventually decided to de-scope fully-qualified slugs, and opt for more simpler and lightweight approach to slugs.
This was primarily due to time constraints, a significant number of breaking changes, and we didn't need fully-qualified slugs for config as code anyway.

Thankfully, there was a bit of ground-work already done for us, as projects already have a concept of slugs. Project slugs are automatically generated based on their names, and re-generated when the name changes. These slugs are used in the project URLs for the web UI.
We were able to re-use this existing project slug generation code, and combine it with a new concept called slug providers

### Introducing Slug Providers

Slug providers live in our document store layer as a decorator. Their job is to ensure that slugs have been set on any documents which need one, and don't already have one, much like how a database will automatically set the ID for any new rows.
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
    var existingDoc = await documentStore.GetBySlugOrNull(d.Slug);
    
    var i = 1;
    var uniqueSlug = slug;
    while (existingDoc is not null)
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
This turned out to be relatively simple, as we could re-use our existing tiny type converter with some added logic for converting between IDs and slugs.
We also didn't need to worry too much about performance anymore, as our new document stores offered improved caching capabilities.

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

It's much easier to understand which environments and worker pool this is referring to now, and we get the added bonus of these slugs being unique.

Because we convert slugs to IDs when reading OCL, this removes the need to account for IDs or names in both the Octopus codebase and API.
This solves the issue of chatty API clients, complicated _"name or ID"_ related logic, and eliminates a whole category of bugs!

### The future of slugs

With much of the groundwork for slugs now completed, we can start adding slugs to more and more resources, and looking into fully-qualified slugs with a fresh set of eyes.
As slugs become more prevelant throughout Octopus, they'll eventually become a more integral part of our integrations, steps, and API.

Happy deployments!
