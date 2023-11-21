---
title: Deprecating the Octo CLI
description: Octopus Deploy will drop support for the Octo CLI
author: eddy.moulton@octopus.com
visibility: public
published: 2023-11-21-1400
metaImage:
bannerImage:
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags:
  - Product
---

Late last year we announced our new and improved Octopus command-line interface (CLI). You can read more about how and why we built it in [John's blog post](https://octopus.com/blog/building-octopus-cli-vnext).

It's now time to retire the Octo CLI. In this post I'll explain why we're deprecating it and what you need to know for your integrations.

## What's wrong with the Octopus CLI (octo)

John says it best in his [state of the Octopus CLI](https://octopus.com/blog/building-octopus-cli-vnext) section from his blog post. The bottom line is that it's limitations have required us to rebuild a cli for more modern workflows. As such we will no longer be providing feature or security updates to the Octo CLI.

## What's new in the Octopus CLI

Since the announcement last year we've been busy adding features to the CLI. Take a look at the [GitHub releases](https://github.com/OctopusDeploy/cli/releases) for a timeline of changes or skip straight to the [docs](https://octopus.com/docs/octopus-rest-api/cli) to see everything you can currently do with the CLI.

## What to expect next

We don't yet have a hard date set to remove the Octo CLI, but rest assured that it will not be a surprise.

We intend to surface deprecation warnings within both the CLI tool and Octopus portal at least 12 months before it's removal.

## Migrating from the Octo CLI

The new Octopus CLI was not designed as a drop in replacement, but most of the functionality provided by the Octo CLI is now supported.

Our public APIs are unlikely to undergo any large changes, so the Octo CLI will continue to work in your existing workflows for the foreseeable future. We recommend making the swap to the new Octopus CLI as soon as practical to enjoy new improvements as they become available.

If you currently rely on functionality that does not have an equivalent in the new CLI then please create an [issue](https://github.com/OctopusDeploy/cli/issues) on our GitHub repository or reach out in our [community Slack](https://oc.to/CommunitySlack).

## Conclusion

The Octo CLI has served us well for many years, but it's almost time to say goodbye.

We want to continue making a CLI that customers love to use so we'd be grateful for any feedback or suggestions through any of our channels.

Happy deployments!
