---
title: Packages in Script Steps 
description: As of Octopus 2018.8 Script Steps will have the ability to reference packages
author: michael.richardson@octopus.com
visibility: private
bannerImage: 
metaImage: 
published: 2019-08-30
tags:
- Scripting
---

In Octopus 2018.8 Script Steps are evolving, and gaining some new super-powers.

## Packages++

We are adding the ability to add package references to the family of script steps:
- `Run a Script`
- `Run an Azure PowerShell Script`
- `Run an AWS CLI Script`

Previously these steps were able to reference a single package which contained the script to be run.  They can now also reference packages which do not contain the script.  Yes, packages, _plural_.  Oh, and _packages_ includes container images. 

### Why?

Custom scripts which 