---
title: Building Config as Code 
description:  
author: michael.richardson@octopus.com
visibility: private
published: 
metaImage: 
bannerImage: 
tags:
 - Product 
---

We have been busy recently building configuration-as-code support for Octopus Deploy.  In this post, we'll talk about 

- Why we're building this feature
- The thinking behind our design decisions 
- The challenges
- The public preview release

But before we do, we should define what we mean by "configuration-as-code". We are referring to a version-controlled, textual representation of an Octopus project. Today, when you configure a project in Octopus, the configuration is stored as records in a relational database. This feature is basically taking some of that data and persisting it as files in a git repository rather than the database.  

## Why config-as-code?

Over the past few years, version-controlling Octopus configuration has been comfortably our most commonly requested feature.  

Often prioritizing features is a trade-off between addressing requests from existing users and building something to appeal to new users. Configuration-as-code was commonly requested by both groups. And we understand why.  The benefits are compelling, and include:
- History: TODO
- Branching: TODO   
- One source of truth: TODO 

## Design

We are certainly not the first product to implement this feature. Many of the tools we integrate with, and compete with, have git integrations. This gave us the opportunity to play with various implementations and to develop a sense of what made the difference between an enjoyable, and not so enjoyable, experience. 

In particular there were a few patterns we want to avoid if possible.

### Anti-pattern 1: Git DB

It is tempting to think of git as "just another database", and to simply swap out one persistance layer for another. So changes are persisted in a git repository, but none of the real power of git is enabled. Branches aren't supported, commit messages can't be supplied, text records may not be human-readable, etc.   

The only benefit of git in this scenario is the historical record, which certainly isn't nothing, but even this is compromised.

Replacing `dbTransaction.Commit()` with `gitRepo.Push()` might be the quickest way, but as a user it is rather disappointing.

### Anti-pattern 2: Baby with the bathwater 

In this anti-pattern, users can opt-in to git integration, but only if they are willing to forfeit other features. 

Having spent the past months building this feature it's very easy to see how this happens, and sometimes it's inevitable. 
For an application built on a relation database, it’s difficult to ensure all the various features still function once a chunk of the application data is no longer stored in the database, and no longer has a single version.

It’s tempting to simply disable them, and convince yourself it's the user's choice. And honestly, in early releases of this feature we will disable some functionality, but wherever possible we are striving hard to ensure the decision to enable git comes with as few compromises as possible. 

### Anti-pattern 3: Obfuscation via abstraction

In this pattern git concepts are abstracted in the application.  An example might be branching exposed as a “draft” metaphor.

There’s nothing necessarily wrong with this, and done right it can be very powerful. But its risky for an application like Octopus, where users are likely to understand git concepts. 

Where possible, we've used git terminology and concepts, rather than trying to hide them under abstractions. 

### YAMSONXML hell

We firmly believe that wanting git integration does not imply wanting to forgo all UI assistance.

Having a human-readable text representation of your config, which you can view history, branch, compare, and merge, is empowering.  Staring at a blinking cursor in an empty text file is not.


- We are building config as code 
- Why 
- Design
- Challenges
- What
- EAP (what's included)