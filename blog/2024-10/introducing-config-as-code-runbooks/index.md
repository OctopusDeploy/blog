---
title: Introducing config as code for Runbooks
description: A brief summary of the post, 170 characters max including spaces.
author: firstname.surname@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - Product
  - Configuration as Code
  - Runbooks
---

We’re excited to announce that config as code for Runbooks is now available for Octopus Cloud customers, and will be available for server customers in the 2024.4 release. 

Config as code support for runbook processes complements our existing support for version-controlled deployment processes. When config as code is enabled, your Runbook process can be tracked in your Git repo next to your related application code or other operation files. When config as code is enabled for a project, you can continue using the Octopus UI as you always have or edit the config files in your favorite editor. 

Config as code for Runbooks brings some changes to the existing Runbooks experience. This blog post explains what comes with config as code for Runbooks and highlights some of the changes you will see. 


## Why config as code for Runbooks?

[Octopus Runbooks](https://octopus.com/docs/runbooks) lets you automate routine and emergency operations tasks, giving you one platform for DevOps automation. Storing your Runbooks in your Git repository alongside your application code, deployment process, and variables unlocks the full power of version control, so you can:

- Make and test changes to your Runbooks alongside your application code changes so they never get out of sync. 
- On top of existing Octopus permissions and audit logs, you can review all changes through pull requests and have a full history of all changes that have been made to your Runbooks. 
- With your Runbooks in Git, it’s easy to run an old version by running the Runbook at a commit or tag. If you make a mistake and need to roll back, you have a full history of all changes and can roll back to any point in time.

## Snapshots changing to Branches

When running a Runbook, you can currently only pick from two different versions: the published snapshot or the latest draft.

When running the published snapshot, we run a version of the Runbook that was snapshot sometime in the past and marked as the published version.
When running the latest draft, we create a new snapshot of the latest process and variables at the time of running every time you run.

This allowed you to make and test changes to your Runbooks without impacting day-to-day operations.

With config as code for Runbooks, you can make and test changes to a Runbook on a branch without impacting the rest of the team, so there is no need for published and draft versions.
Once your changes have been tested and reviewed simply merge them into your default branch to make them available for the rest of the team.

You can also run an old version of a Runbook from a commit or tag at any time - there is no need to go searching through old snapshots to try and find one that works.


### How will Runbook Permissions work?

If you are currently making use of the existing built-in Runbook Producer and Runbook Consumer roles, these will still apply to config as code for Runbooks.

Runbook producers can run runbooks on any branch, commit, or tag.
Runbook consumers can view a Runbook on any branch but can only run the latest version of the Runbook from the default branch.


## Conclusion

Config as code for Runbooks is currently rolling out to all our Cloud Customers, and will be available for server customers as part of the 2024.4 release. If you want to use config as code for Runbooks, you will need to migrate your existing runbooks using our simple migration tool  a prompt will appear when available in your instance. Make sure you keep an eye out for our updates for more about what to expect with our most anticipated feature.

Happy deployments!