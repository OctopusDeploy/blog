---
title: Deprecating the Octo CLI
description: Learn why Octopus is dropping support for the Octo CLI and what's next.
author: eddy.moulton@octopus.com
visibility: public
published: 2023-11-29-1400
metaImage:
bannerImage:
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags:
  - Product
---

Late last year, we announced our new and improved Octopus command-line interface (CLI Octopus). You can read more about how and why we built it in John Bristowe's blog post, [Building the Octopus CLI vNext](https://octopus.com/blog/building-octopus-cli-vnext).

It's now time to retire the old Octo CLI (octo). In this post I explain why we're deprecating it and what you need to know for your integrations.

## What's wrong with the Octo CLI?

The Octo CLI has a range of limitations that you can read about in [John's blog post](https://octopus.com/blog/building-octopus-cli-vnext#the-state-of-the-octopus-cli-octo). These limitations mean we need to rebuild a CLI for more modern workflows. As such, we'll no longer be providing feature or security updates to the Octo CLI.

## What's new in the new Octopus CLI?

Since the announcement last year, we've been busy adding features to the new CLI. Take a look at the [GitHub releases](https://github.com/OctopusDeploy/cli/releases) for a timeline of changes or skip straight to the [docs](https://octopus.com/docs/octopus-rest-api/cli) to see everything you can do with the CLI.

## What to expect next

We haven't set a hard date to remove the Octo CLI, but it won't be a surprise.

We intend to surface deprecation warnings in both the CLI tool and the Octopus portal at least 12 months before it's removal.

## Migrating from the Octo CLI

The new Octopus CLI wasn't designed as a drop in replacement, but most of the functionality provided by the Octo CLI is now supported.

Our public APIs are unlikely to undergo any large changes, so the Octo CLI will continue to work in your existing workflows for the foreseeable future. We recommend making the swap to the new Octopus CLI as soon as practical to enjoy new improvements as they become available.

If you currently rely on functionality that doesn't have an equivalent in the new CLI, please either:

- Create an [issue in our GitHub repository](https://github.com/OctopusDeploy/cli/issues)
- Reach out in our [Community Slack](https://oc.to/CommunitySlack)

## Conclusion

The Octo CLI has served us well for many years, but it's almost time to say goodbye.

We want to continue making a CLI that customers love to use, so we'd be grateful for any feedback or suggestions. You can leave a comment below, contact us on [Community Slack](https://oc.to/CommunitySlack), or create an issue in [GitHub](https://github.com/OctopusDeploy/cli/issues).

Happy deployments!