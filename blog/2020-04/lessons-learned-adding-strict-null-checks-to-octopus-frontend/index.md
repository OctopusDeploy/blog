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

## Possible Options

### We went with non-null-assertion operator

### Caveats around this approach

## Lessons learned
---> Union of types can be mutually exclusive i.e. Record union with object with optional props
---> Indexing into types can get tricky with undefined in the mix
---> Strict nulls also means stricter checks in functions (Ref is a good example)
---> Types are more strict, so we had to separate creation of new resources from existing resources since in our case the shapes were different i.e. links in existing resources vs not having to provide it when creating new
---> More effectively use types such as unions in order to be able to narrow types
---> Rules of thumb, try and check for a type or null only once. Narrowing types is your best friend (introduce the general pattern, then an example of how we applied it)
---> Sane defaults can help quite a bit
---> Try and work with the most restrictive type you can in most scenarios

## Conclusion
(start with strict nulls is much easier than enabling it later)