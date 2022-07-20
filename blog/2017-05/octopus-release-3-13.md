---
title: "Octopus May Release 3.13"
description: This month's release brings some exciting new features including support for Azure Service Fabric, HSTS, optional lifecycles and performance improvements, among other things!
author: mark.siedle@octopus.com
visibility: public
published: 2017-05-03
bannerImage: shipping-3-13-blogimage.png
bannerImageAlt: Octopus 3.13 release announcement
tags:
 - Product
---

![Octopus 3.13 release announcement](shipping-3-13-blogimage.png)

This month's release brings some exciting new features including support for Azure Service Fabric, HSTS, optional lifecycles and performance improvements, among other things!

## In this post

!toc

## Release Tour

<iframe width="560" height="315" src="https://www.youtube.com/embed/LljoIA8wtHQ" frameborder="0" allowfullscreen></iframe>

## Introducing Azure Service Fabric support

We are excited to announce that Octopus now includes first-class support for [Deploying Azure Service Fabric applications](https://octopus.com/docs/deployments/azure/service-fabric/deploying-a-package-to-a-service-fabric-cluster).

Since the [RFC](https://octopus.com/blog/rfc-azure-service-fabric) earlier this year, we've been busy creating new step templates to help you connect to and deploy your Service Fabric cluster applications.

These new Service Fabric steps can now assist you with:

- Deploying a Service Fabric App ([learn more](https://octopus.com/docs/deployments/azure/service-fabric/deploying-a-package-to-a-service-fabric-cluster)
- Running a Service Fabric SDK PowerShell Script ([learn more](https://octopus.com/docs/deployments/custom-scripts/service-fabric-powershell-scripts))

Both steps require connection to a cluster. As such, we've included support for security modes including unsecure, Client Certificates and Azure Active Directory.

:::hint
**Service Fabric SDK**
Due to Service Fabric dependencies, you will need to manually install the [Service Fabric SDK](https://g.octopushq.com/ServiceFabricSdkDownload) onto your Octopus Server.  Then you can then start using Octopus to help orchestrate your Service Fabric application deployments.
:::

## Want to learn more?

You can learn more about these new features from our main [Deploying to Service Fabric](https://octopus.com/docs/deployments/azure/service-fabric) documentation. 

## HTTP Strict-Transport-Security (HSTS)

HTTP Strict Transport Security is an HTTP header that can be used to tell the web browser that it should only ever communicate with the website using HTTPS, even if the user tries to use HTTP. This can substantially lessen your attack surface, and is frequently recommended by security professionals. 

We can now send this header on demand, but as there are some potential complexations, it is not enabled by default. If you have your Octopus Server exposed on the internet, we recommend [reading up on and enabling HSTS](https://octopus.com/docs/how-to/expose-the-octopus-web-portal-over-https#HSTS) if you can.

## Optional lifecycle Phases

Knocking off another high ranking [UserVoice suggestion](https://octopusdeploy.uservoice.com/forums/170787-general/suggestions/8475958-lifecycle-optional-phase-or-optional-environment) from our backlog, you can now create optional phases in your lifecycle that can be skipped during progression. This feature will help for those cases where you want to have the freedom to deploy your release to a set of environments, without holding up the deployment from continuing. [Channels](https://octopus.com/docs/deployments/patterns/branching) work well when this behaviour is known up-front and is part of a standard release pipeline, for example always pushing a hotfix release straight to UAT, but this approach is too rigid for the more fluid set of rules that optional phases functionality brings.

Learn more about [optional lifecycle phases](https://octopus.com/docs/releases/lifecycles).

## Browser caching

Loading the the dashboard can be quite a data intensive operation for the Octopus Server to perform. It potentially needs to extract all the releases and deployments for your projects, compare, sort and filter them, then serialize and return to the browser to render. On large instances this can take many seconds to complete. Each time this request is made from the client this ties up server resources for something that more often than not (considering it currently updates every 5 seconds or so) may not have even changed. Instead the portal will now cache responses for some high cost queries and only reload the data if new events have taken place on the Server. Although this feature can be disabled, it is expected that this will make the Server more responsive for all users.

## Failing a script with a message

The message on the deployment overview can now be customised, refer to [failing a script with a message](https://octopus.com/docs/deployments/custom-scripts/logging-messages-in-scripts)

## Modify Task State

Have you ever deployed to Production only to have your last step "Email release party invites!" fail?  Or maybe you deployed sucessfully but after some QA decided to roll back. Now you can modify the state of a task.  When a task has completed with the state `Success`, `Failed` or `Canceled` you can edit the state from the task screen by providing the new task state and the reason for the change.  Once submitted, the task state will be updated and an entry in the task history will contain an audit entry with the change.  A new permission called `TaskEdit` is required to perform this action.  By default the `TaskEdit` permission has only been granted to the built-in Administrators team.

## Channel-indexed version-template variables

e.g.
```
#{Octopus.Version.Channel[MyChannel].LastMajor}
```

Currently, when using a template for calculating release versions many variables are made available. e.g. 

```
#{Octopus.Version.LastMajor}
#{Octopus.Version.NextMajor}
#{Octopus.Version.LastMinor}
...
``` 

When using [Channels](https://octopus.com/docs/releases/channels), corresponding variables are made available for the _current_ channel. e.g.

```
#{Octopus.Version.Channel.LastMajor}
#{Octopus.Version.Channel.NextMajor}
#{Octopus.Version.Channel.LastMinor}
...
``` 

The piece that had been missing was the ability to reference the version components of _other_ channels (i.e. not the channel of the release being created).

For example, your project may have the channels: `Release`, `PreRelease`, and `HotFix`. You will now be able to define a version template such as:

```
#{if IsRelease}
  #{Octopus.Version.Channel.LastMajor}.#{Octopus.Version.Channel.NextMinor}.0
#{/if}
#{if IsPreRelease}
  #{Octopus.Version.Channel[Release].LastMajor}.#{Octopus.Version.Channel[Release].NextMinor}.0#{Octopus.Version.Channel.NextSuffix}
#{/if}
#{if IsHotFix}
  #{Octopus.Version.Channel[Release].LastMajor}.#{Octopus.Version.Channel[Release].LastMinor}.#{Octopus.Version.Channel.NextPatch}
#{/if}
``` 

The use of the channel-indexed variables such as `#{Octopus.Version.Channel[Release].LastMajor` allows you to reference the previous\next version components of other channels.

N.B. This template depends on defining the channel-scoped variables `IsRelease`, `IsPreRelease`, and `IsHotFix`.  

## Upgrading

All of the usual [steps for upgrading Octopus Deploy](https://octopus.com/docs/administration/upgrading) apply. Please note there is a minor breaking change in this release around the `show-configuration` command. Please see the [release notes](https://octopus.com/downloads/compare?to=3.13.0) for further information.

## Wrap Up

That’s it for this month. We hope you enjoy the latest features and our new release. Feel free to leave us a comment and let us know what you think!  Happy deployments!
