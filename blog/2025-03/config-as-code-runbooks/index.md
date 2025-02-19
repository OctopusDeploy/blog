---
title: Configuration as Code for Runbooks
description: Learn about the new Config as Code for Runbooks.
author: harriet.alexander@octopus.com
visibility: public
published: 2025-03-03-1400
metaImage: img-configascode-runbooks-2024.png
bannerImage: img-configascode-runbooks-2024.png
bannerImageAlt: Graphic of a woman sitting on a large stack of books while working on laptop, surrounded by large, three-dimensional curly braces.
isFeatured: false
tags: 
  - Product
  - Configuration as Code
  - Runbooks
---

Config as Code for Runbooks is officially landing as part of Octopus 2025.1. This feature gives you a seamless, version-controlled way to manage your runbooks. 

You can run, create, and track your runbooks in your chosen Git repository, bringing your runbooks together with your deployment process. This means fewer places to make changes and provides more control over your workflows.

## Why Config as Code for Runbooks?

[Octopus Runbooks] (https://octopus.com/docs/runbooks) automates routine and emergency operations tasks. Storing your runbooks in your Git repo with your application code, deployment process, and non-sensitive variables unlocks the full power of version control.

- Store your runbooks in your Git repo to stay in sync. You can make and test changes alongside your app code, so your code and operational processes never get out of sync.
- Store your runbooks in Git to version-control your operational tasks. This way, you can manage versions with minimal effort. If you need to roll back or retrieve an earlier configuration, you can run the runbook for a specific commit or tag.
- Access the complete history for audibility. You can review all changes via pull requests, providing a full history of all changes to your runbooks.
- Rollback easily. Mistakes happen! But with version control in place, it's easier than ever to revert to a previous version of your runbooks. The complete history lets you track and manage past configurations.

## How to use Config as Code for Runbooks

Version control is a requirement for Config as Code for Runbooks. 

- If your projects are version-controlled, the setup and migration is quick and easy. You'll also find using Config as Code for Runbooks natural. Following Git principles will make the process straightforward.
- If you're not using version control yet, you can enable it and start using Config as Code for Runbooks. [See our docs for a guide on enabling version control for your project](https://octopus.com/docs/projects/version-control/converting).


### Migrating existing runbooks

If you use Octopus Runbooks, we have a tool to help you migrate to Config as Code for Runbooks.

- Migration process: When you start the migration, Octopus moves your published runbooks to a new folder, `/runbooks`, in your chosen repository. This folder stores the published snapshot and configuration from the runbook.
- Octopus migrates any draft versions of your runbook to a separate sub-folder `/runbooks/migrated-draft`. You won't see these in the UI, but you can access them by moving them into the main runbooks folder using a pull request (PR).
- History retention: You don't need to worry about losing your history. The UI will still show your existing runbook runs.

![Screen recording of Runbooks migration process. Starting on the Runbook list, the user opens the wizard, reviews the changes, creates a new branch named feat/migrate-runbooks, uses the default commit message, reviews the migration and completes it. Once completed, the Runbooks list screen is automatically refreshed showing a branch selector and an alert showing there are un-migrated drafts](configascode.gif) 

## New in Config as Code for Runbooks

There are a few tweaks to how Runbooks work in the Config as Code setup. We made these adjustments to improve functionality and streamline your workflow. If you're curious about these changes, [read our post, Introducing Config as Code for Runbooks](https://octopus.com/blog/introducing-config-as-code-runbooks).

:::hint
These changes apply exclusively to Config as Code for Runbooks. If you're not using version control, they won't impact your runbooks.
:::

## Conclusion

Config as Code for Runbooks has been one of our most highly requested features. Our community wanted it, and we're excited to deliver it. 

Putting runbooks in your Git repos with your app code simplifies DevOps workflow management. It ensures everything is version-controlled and fully auditable. To see it in action, you can [watch our webinar, Introducing Config as Code for Runbooks](https://www.youtube.com/watch?v=oRWKDUzYBYA).

Config as Code for Runbooks has rolled out to Cloud customers and will be available soon for self-hosted customers in the 2025.1 release. We can't wait for you to try it out and experience the benefits for yourself.

Happy deployments!