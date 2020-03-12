---
title: "Lessons learned porting Octopus Server to .NET Core 3.1"
description: "TODO"
author: rob.pearson@octopus.com
visibility: private
bannerImage: 
metaImage: 
published: 2020-02-05
tags:
- Engineering
---

Intro - BLUF

## History

## Why 

## Top 3 Lessons Learned

### 1. Platform specific differeneces

Context

Problem. 

* MARS <- SQL server - performance problems on *nix
* Moving everything out of the registry and storing configuration differntly. <- Cloud v2
* Ask shannon about perf w/ something on Auth/Azure Linux - Active Directory 
Windows auth on Linux. We needed granular control over auth providers not just all or nothing. 

Solution.

* MARS - Turn off support. Rely 
We open two connections and pick the one that suits. 

5 areas that still have MARS. 

Still workin with microsoft to resolve but we don't have a concrete timeline. 

* Windows auth on Linux.
Routing - Run two websites listening on the same URL and port and handles things nicely. - Found on the Internet. 

Examples. 

### 2. Learning how to debug .NET Core on Linux and Docker

Start 

### 3. Shipping self-contained packages. 

Talk about supporting deploying self-contained packages

Context

Why? 

Benefits.

Caveats - tradeoff - drop support for older systems. Better for staying modern and innovating etc. Link to other blog post. 
Competitor started today, they wouldn't support old operation systems. 

(stay relevant)

Business decision to move forward. 

Docker is self-contained thus it's a a very supportable platform.

## Conclusion

Recap BLUF.
