---
title: Importing Kubernetes YAML in Octopus
description: Learn how to import existing Kubernetes YAML into Octopus steps
author: matthew.casperson@octopus.com
visibility: private
published: 2999-01-01
metaImage: 
bannerImage: 
tags:
 - Octopus
---

If you have been using Kubernetes for some time outside of Octopus, you likely have existing YAML resource definitions. Migrating this YAML into Octopus is easy thanks to a new feature introduced in Octopus 2020.2, giving you the best of both world with the ability to import, export and edit raw YAML while having your Kubernetes resources managed in an opinionated way by Octopus.

In this blog post we'll learn how to migrate existing YAML definition into an Octopus deployment.