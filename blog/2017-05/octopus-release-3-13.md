---
title: "Octopus May Release 3.13"
description: TODO
author: mark.siedle@octopus.com
visibility: private
tags:
 - New Release
 - Azure Service Fabric
---

![Octopus 3.13 release announcement](shipping-3-13-blogimage.png)

This month's release brings some exciting new features including support for Azure Service Fabric, HSTS, optional lifecycles and performance improvements, among other things!

## In this post

!toc

## Release Tour

<iframe width="560" height="315" src="#" frameborder="0" allowfullscreen></iframe>

## Introducing Azure Service Fabric support

We are excited to announce that Octopus now includes first-class support for [Deploying Azure Service Fabric applications](https://octopus.com/docs/deploying-applications/deploying-to-service-fabric).

Since the [RFC](https://octopus.com/blog/rfc-azure-service-fabric) earlier this year, we've been busy creating new step templates to help you connect to and deploy your Service Fabric cluster applications.

These new Service Fabric steps can now assist you with:

- Deploying a Service Fabric App ([learn more](https://octopus.com/docs/deploying-applications/deploying-to-service-fabric/deploying-a-package-to-a-service-fabric-cluster))
- Running a Service Fabric SDK PowerShell Script ([learn more](https://octopus.com/docs/deploying-applications/custom-scripts/service-fabric-powershell-scripts))

Both steps require connection to a cluster. As such, we've included support for security modes including unsecure, Client Certificates and Azure Active Directory.

:::hint
**Service Fabric SDK**
Due to Service Fabric dependencies, you will need to manually install the [Service Fabric SDK](https://g.octopushq.com/ServiceFabricSdkDownload) onto your Octopus Server. Then you can then start using Octopus to help orchestrate your Service Fabric application deployments.
:::

## Want to learn more?

You can learn more about these new features from our main [Deploying to Service Fabric](https://octopus.com/docs/deploying-applications/deploying-to-service-fabric) documentation. 

We also have a new guide explaining [Continuous Integration for Service Fabric](https://octopus.com/docs/guides/service-fabric) where you can learn how Octopus Deploy fits into a Continuous Deployment pipeline for you Service Fabric applications.

## HTTP Strict-Transport-Security (HSTS)

HTTP Strict Transport Security is an HTTP header that can be used to tell the web browser that it should only ever communicate with the website using HTTPS, even if the user tries to use HTTP. This can substantially lessen your attack surface, and is frequently recommended by security professionals. 

We can now send this header on demand, but as there are some potential complexations, it is not enabled by default. If you have your Octopus Server exposed on the internet, we recommend [reading up on and enabling HSTS](https://octopus.com/docs/how-to/expose-the-octopus-web-portal-over-https#HSTS) if you can.

## Optional lifecycles

TODO

## Browser caching

TODO

## Failing a script with a message

The message on the deployment overview can now be customised, refer to [failing a script with a message](https://octopus.com/docs/deploying-applications/custom-scripts#failing-a-script-with-a-message)

## Modify Task State
Have you ever deployed to Production only to have your last step "Email release party invites!" fail?  Or maybe you deployed sucessfully but after some QA decided to roll back. Now you can modify the state of a task.  When a task has completed with the state `Success`, `Failed` or `Canceled` you can edit the state from the task screen by providing the new task state and the reason for the change.  Once submitted, the task state will be updated and an entry in the task history will contain an audit entry with the change.  A new permission called `TaskEdit` is required to perform this action.  By default the `TaskEdit` permission has only been granted to the built-in Administrators team.

## Channel-indexed version-template variables

e.g.
```
#{Octopus.Version.Channel[MyChannel].LastMajor
```

Currently, when using a template for calculating release versions many variables are made available. e.g. 

```
#{Octopus.Version.LastMajor}
#{Octopus.Version.NextMajor}
#{Octopus.Version.LastMinor}
...
``` 

When using Channels, corresponding variables are made available for the _current_ channel. e.g.

```
#{Octopus.Version.Channel.LastMajor}
#{Octopus.Version.Channel.NextMajor}
#{Octopus.Version.Channel.LastMinor}
...
``` 

The piece that had been missing was the inability to reference the version components of _other_ channels (i.e. not the channel of the release being created).

For example, you may have three channels: `Release`, `PreRelease`, and `HotFix`. You will now be able to define a version template such as:

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

Note the use of the channel-indexed variables such as `#{Octopus.Version.Channel[Release].LastMajor`.
This template depends on defining the channel-scoped variables `IsRelease`, `IsPreRelease`, and `IsHotFix`.  

## Upgrading

All of the usual [steps for upgrading Octopus Deploy](https://octopus.com/docs/administration/upgrading) apply. Please note there is a minor breaking change in this release around the `show-configuration` command. Please see the [release notes](https://octopus.com/downloads/compare?to=3.13.0) for further information.

## Wrap Up

That’s it for this month. We hope you enjoy the latest features and our new release. Feel free to leave us a comment and let us know what you think!  Happy deployments!
