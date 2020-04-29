---
title: Lessons learned adding strict null checks to the Octopus front-end
description: TODO
author: rob.pearson@octopus.com
visibility: private
published: 3020-04-01
metaImage:
bannerImage:
tags:
  - Engineering
  - TypeScript
---

Introduction

## Why

Nulls are often said to be the billion dollar mistake. They often creep up when you least expect them and in some pretty interesting ways. The problem isn't in the representation of null itself, it's more that we specifically
forget to deal with the `null` case. It's for this reason that some languages completely shy away from the concept of null and choose to represent the concept in a type safe way using the `Option` monad. Functional programming
isn't always an easy sell and in some cases we may still have legacy code bases. This is where `strictNullChecks` compiler flag that typescript provides helps. This single switch allows typescript to treat `null` and `undefined`
as separate types which forces those cases to be handled.

## Possible Options

Enabling strict nulls on an existing codebase should be as simple as flipping a switch, right? Unfortunately it's not quite that simple. In our case, simply enabling the flag resulted in around 5k errors, which isn't a trivial
amount to work through. This means we had to determine the best course of action for us to enable strict nulls. Let's look at a couple of the options:

### Do it all in one go

There may be a number of reasons for being able to convert everything in one pass. It may mean that you already wrote perfect strict null compliant code to begin, which is very unlikely or you have a pretty small code base. In our
particular case however, the sheer magnitude of the changes we would have to make along with all the unknowns made this an absolute no-starter. It may sound like it would be achievable if you roll up your sleeves and do it, however
we found that unless you apply surgical precision typing changes you are very likely to introduce many more errors for every typing change that you make. It's hardly easy to estimate when numbers keep changing under you.

### Segragate our frontend codebase

With all the rage around monorepos and typescript supporting project references this is rather an attractive option. This would effectively allow us to take segments of our codebase and separate these into separate projects
allowing us to enable strict nulls for each individual segment incrementally. At first glance this option is extremely attractive, however the tooling around this didn't feel quite complete yet. There are a number of issues
surrounding project reference support in `fork-ts-checker-webpack-plugin`, while project references don't make much sense for react based libraries at the moment for example. This is still something we would like to revisit in the
future, but isn't something we had the appetite for as part of converting to strict nulls.

### Multiple tsconfigs

We could have introduced multiple `tsconfig.json` files, one which excludes strict nulls and another which includes it. During development we could have enabled strict nulls while we used a different config excluding strict nulls
for production builds. The idea behind this is to apply downards pressure over time on non-compliant code, however it would also mean that you would possilby have 7000 odd errors constantly hitting you in the face. This felt less than
ideal. A variation on this, would be to let the IDE use the tsconfig with strict nulls enabled which keeps all the errors out of the console.

### Multiple tsconfigs with assertion

In this strategy you would disable strict nulls for your normal development and production builds. You would then add an additional tsconfig with strict nulls enabled which contains an explicit list of files to be compliant. This would
also require an additional step in CI builds to run a `tsc` check against the tsconfig that includes the strict null checks flag. This ensures that additional errors introduced will be picked up, but you may not necessarily see these
during normal development. This may work well when intentionally contributing significant engineering effort towards the endeavour over a shorter period of time.

### The non-null-assertion operator

Typescript has the non null assertion operator (`!`) which acts as a means to tell typescript that something is not null within a particular scope. This allows typing issues surrounding `null` and `undefined` to be supressed on a case by case basis,
but requires the supressions be done for every error when strict nulls is enabled. Enabling linting rules to prevent the operator's usage and supressing these on a file by file basis is also beneficial to prevent the spread of the operator. This approach
allows you to effectively enable strict-nulls at a file level and the associated work to get there can be predictablly estimated.

We landed on this as our preferred option as we wanted to be able to implement strict nulls in a predictable manner with as little risk with as little imapct as possible.

### Caveats around this approach

There are some real standout negatives to the chosen approach which is well worth calling out.

- It requires a lot of mechanical supressions
- It can lead to a partially implemented strict null code base, which never ends up being fully completed without focused effort
- Non-null assertions can spread if not kept in check
- You can end up with some nasty code smells by using the non-null assertion operator in this fashion e.g:

```
    this.state = {
        variables: null!;
    }
```

We also knew that we would be putting in more effort in the long run to address these issues, so we felt that the pros outweighed the cons in our case.

## Lessons learned

As part of our conversion process, we didn't just leave things as they were after applying non-null assertions everywhere. We also wanted to take a feature area and convert it to be strict null compliant. We felt that we
wanted to get a better understanding of the types of changes we would need to make and determine whether there are any patterns that we could apply to make our lives easier. We wanted to share some of these learnings with
others in case you find it useful.

### Union of types can be mutually exclusive

Typescript with strict nulls will treat treat `null` and `undefined` as different types and it may be surprising at first.

---> Indexing into types can get tricky with undefined in the mix
---> Strict nulls also means stricter checks in functions (Ref is a good example)
---> Types are more strict, so we had to separate creation of new resources from existing resources since in our case the shapes were different i.e. links in existing resources vs not having to provide it when creating new
---> More effectively use types such as unions in order to be able to narrow types
---> Rules of thumb, try and check for a type or null only once. Narrowing types is your best friend (introduce the general pattern, then an example of how we applied it)
---> Sane defaults can help quite a bit
---> Try and work with the most restrictive type you can in most scenarios

## Conclusion

(start with strict nulls is much easier than enabling it later)
