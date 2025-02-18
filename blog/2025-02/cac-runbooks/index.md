---
title: Introducing Config as Code for Runbooks
description: Everything you need to know about the new Config as Code for Runbooks
author: harriet.alexander@octopus.com
visibility: private
published: 2025-02-17-1400
metaImage: cacrunbooks image.png
bannerImage: cacrunbooks image.png
bannerImageAlt: Graphic of a woman sitting on a large stack of books while working on laptop, surrounded by large, 3 dimensional curly braces
isFeatured: false
tags: 
  - product, Configuration as Code, Runbooks
---

### Introducing Config as Code for Runbooks
We are excited to announce a feature that many have anticipated for a long time: Config as Code for Runbooks. This feature adds Config as Code support to Runbooks it gives you a seamless, version-controlled way to manage your workflows. This new addition lets you run, create, and track your Runbooks in your chosen Git repository bringing your runbooks together with your deployment process. This means fewer places to make changes and provides more control over your workflows. 

## Why Config as Code for Runbooks?

[Octopus Runbooks] (https://octopus.com/docs/runbooks) automates routine and emergency operations tasks. Storing your Runbooks in your Git repo with your application code, deployment process, and non-sensitive variables unlocks the full power of version control by:
* Stay in Sync: Store your Runbooks in your Git repo. This lets you make and test changes alongside your app code. This ensures that your code and operational processes never get out of sync.
* Version Control for Operational Tasks: Store your Runbooks in Git. This way, you can manage versions with minimal effort. If you need to roll back or retrieve an earlier configuration, you can run the Runbook for a specific commit or tag.
* Complete History and Audibility: You can review all changes via pull requests providing a complete history of all changes to your Runbooks.
* Easy Rollbacks: Mistakes happen! But with version control in place, it's easier than ever to revert to an old version of your Runbooks. The complete history lets you track and manage past configurations.

## How do I use Config as Code for Runbooks?
* Version Control is a Requirement: If your projects are version-controlled, the setup and migration will be smooth and easy. If you're not using version control yet, don't worry! [See our docs for a guide on enabling version control for your project](https://octopus.com/docs/projects/version-control/converting)
* Familiar Workflow: If you're used to version-controlled projects, you'll find Config as Code for Runbooks natural. Following Git principles will make the process straightforward.

### Migrating Existing Runbooks
If you use Octopus's Runbooks, we have a tool to help you migrate to Config as Code for Runbooks.
* Migration Process: When you start the migration, Octopus will move your published Runbooks to a new folder, `/runbooks`, in your chosen repository. This folder will store the published snapshot and configuration from the Runbook.
* We will migrate any draft versions of your Runbook to a separate sub-folder `/runbooks/migrated-draft`. You won't see these in the UI. But you can access them by moving them into the main Runbooks folder using a pull request (PR).
* History Retention: Don't worry about losing your history. The UI will still show your existing Runbook runs.

![Screen recording of Runbooks migration process. Starting on the Runbook list, the user opens the wizard, reviews the changes, creates a new branch named feat/migrate-runbooks, uses the default commit message, reviews the migration and completes it. Once completed, the Runbooks list screen is automatically refreshed showing a branch selector and an alert showing there are un-migrated drafts] (2025-02-11 08.07.28.gif) 

### What's new in Config as Code for Runbooks?
As with any new feature, there are a few tweaks to how Runbooks work within the Config as Code setup. We've made these adjustments to enhance functionality and streamline your workflow. If you're curious about these changes, [read our recent blog post] (https://octopus.com/blog/introducing-config-as-code-runbooks).
*Note*: These changes apply exclusively to Config as Code for Runbooks. If you're not using version control, they won't impact your Runbooks.

## Conclusion

Config as Code for Runbooks is a highly requested feature. Our community wanted it, and we're excited to deliver it. Putting Runbooks in your Git repos with your app code simplifies DevOps workflow management. This feature will streamline your tasks. It will ensure everything is version-controlled and fully auditable.
This feature is now rolling out to Cloud customers and will be available in the 2025.1 release. We can't wait for you to try it out and experience the benefits for yourself.

Happy deployments!
