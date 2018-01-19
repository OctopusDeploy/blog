---
title: "Octopus January Release 2018.1"
description: What's new in Octopus 2018.1
author: shaun.marx@octopus.com
visibility: private
metaImage: metaimage-shipping-2018-1.png
bannerImage: blogimage-shipping-2018-1.png
published: 2018-01-21
tags:
 - New Releases
---

This January release of Octopus Deploy is primarily a security release driven by [Octopus Cloud](https://octopus.com/cloud/register-interest) and will also benefit all of our customers. We highly recommend upgrading.

You'll also notice our new versioning strategy: what would normally have been Octopus `4.2` is actually `2018.1`.

## In this post

!toc

## Octopus Cloud alpha

One of our big drivers for this year is to bring our hosted Octopus offering to market, called **Octopus Cloud**. If you are interested in being part of our closed alpha program, you can [register your interest today](https://octopus.com/cloud/register-interest)! The closed alpha will kick off very soon, followed by an open beta, the official launch, and later in the year we will be launching our Data Centre edition.

Learn more about [Octopus Cloud](https://octopus.com/purchase).

## We've changed our versioning strategy

Read more about [why we changed](version-change-2018.md) which also dives into our evolution as a software product company.

## Security enhancements

In preparation for [Octopus Cloud](https://octopus.com/purchase) we've performed penetration testing and a security audit. No major issues were found but uncovered a few smaller issues we wanted to fix. You may have seen several security fixes in recent patches. This feature release rolls up all of those patches, along with some extra enhancements you can use to make your Octopus Server more secure than ever.

### New built-in roles and permissions

Octopus ships with several built-in [teams and user roles](https://octopus.com/docs/administration/managing-users-and-teams), one of which is the `Octopus Administrators` team with the `System Administrator` role granting the rights to do anything and everything in your Octopus installation.

In Octopus `2018.1` we have affectively split the `Octopus Administrators` team and its `System Administrator` user role into two parts:

- We've kept the existing `Octopus Administrators` team and `System Administrator` role which is all about configuring the Octopus Server and how it is hosted
- We've added a new `Octopus Managers` team and `System Manager` role which is all about configuring your users, teams, projects, environments, etc

This makes a lot of sense for [Octopus Cloud](https://octopus.com/purchase):

- Our team needs to configure how your Octopus Server is hosted, but shouldn't be able to see anything else - we'll be members of the `Octopus Administrators` team.
- We don't want you to inadvertently break your Octopus Server by changing its hosting configuration - you'll be added to the `Octopus Managers` team.

This new division makes sense for larger installations of Octopus, where you want to have a clearer distinction between teams and their responsibilities.

#### New permissions

The underpinning of these changes are a series of new permissions which you may use in your own Octopus installation:

- `UserEdit` was added to fill a gap in our existing permission structure for editing users directly - this was previously required the `AdministerSystem` permission
- `FeatureView` and `FeatureEdit` for independent control over who can see which Octopus features are enabled and who can enable/disable features
- `ConfigurationView` and `ConfigurationEdit` controls who can change hosting-related configuration which all exist under the {{Configuration>Settings}} page
- `SmtpSettingsView` and `SmtpSettingsEdit` controls who can configure an SMTP server for sending email

#### Upgrade experience

When you upgrade your existing Octopus Server, or start a brand new installation of Octopus Server:

- the `System Administrator` user role will be automatically granted the new permissions (this is the same process we've used when adding new permissions in the past)
- the `Octopus Managers` team and `System Manager` role will be automatically created and granted the appropriate permissions

### Actively preventing escalation of privileges (CVE-2018-5706)

[CVE-2018-5706](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-5706)

The built-in user roles are sufficient for most common scenarios. In larger scale installations of Octopus people tend to tailor these user roles, or create their own user roles from scratch. The permissions system in Octopus is very powerful, but with great power comes additional complexity... and great responsibility. In earlier versions of Octopus, if you configured these roles incorrectly you might have accidentally granted a team the ability to escalate their own privileges.

Octopus `2018.1` has an additional layer of security which actively prevents any user from escalating their own or another user's privileges beyond the scope of their current scope of privileges.

Learn more about [this issue](https://github.com/OctopusDeploy/Issues/issues/4167).

### Running tasks on the Octopus Server as a different user

Prior to Octopus `3.0` you required a Tentacle somewhere to do work as part of a deployment. If you wanted to deploy a web site to Azure, you would need to configure a Tentacle as a deployment target and use it as a jump box, when all you needed to do was push your package to Azure and call some APIs. That Tentacle wasn't really a deployment target at all.

In Octopus `3.0` we added the concept of a **worker** into Octopus Server. It's not a concept we've talked about much to date because it's an underlying technology that's built into Octopus Server and just does what it says on the box: work. Now you could install Octopus Server and deploy a web site to Azure in minutes all without any Tentacles involved.

In Octopus `3.3` we added a feature which lets you [run deployment steps on the Octopus Server](https://octopus.com/docs/deployment-process/how-to-run-steps-on-the-octopus-server) which again removed a lot of friction for scenarios other than deploying to Azure.

However, this convenience came at a cost: security. By default Octopus Server runs as the highly privileged `Local System` account on Windows. We typically recommend running Octopus Server as a different account, either a User or Managed Service Account (MSA), so you can grant specific privileges to that account.

#### Now: Built-in worker as a different user

In Octopus `2018.1` you can configure the built-in worker to execute tasks as a different user account. This user account can be a down-level account with very restricted privileges.

```plaintext
Octopus.Server.exe builtin-worker --username=OctopusWorker --password=XXXXXXXXXX
```

All tasks which execute on the Octopus Server will run as that user account. The only gotcha is that the user account running the Octopus Server needs the correct privileges to launch processes as the built-in worker user and impersonate the built-in worker user.

We outline these requirements and how to configure your Octopus Server in our documentation TBD!

#### Future: External workers

We [talked about workers briefly in our 2018 roadmap](roadmap-2018.md#scalability-and-continual-improvements) and will be talking in more detail with you all about this in the near future. In summary, workers will be an evolution of `Run on Server` which will let you delegate tasks in a deployment to a worker which may be external to your Octopus Server. We still plan to ship a built-in worker which you can configure as we described earlier, or disable it altogether as you see fit.

## Breaking Changes

We've made two behavioral changes to Octopus which may effect certain customers.

- [Package acquisition can sometimes be too parallel](https://github.com/OctopusDeploy/Issues/issues/3974)
- [Auto machine removal timing does not take into account if Octopus Server is offline](https://github.com/OctopusDeploy/Issues/issues/3924)

## Upgrading

As usual [steps for upgrading Octopus Deploy](https://octopus.com/docs/administration/upgrading) apply. Please see the [release notes](https://octopus.com/downloads/compare?to=2018.1.0) for further information.

## Wrap up

That’s it for this month. We hope you had a fantastic festive season and find the new features useful. Feel free leave us a comment and let us know what you think! Go forth and deploy!