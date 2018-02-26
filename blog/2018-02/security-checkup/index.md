---
title: Security checkup for your Octopus
description: Take a few minutes to give your Octopus a security health check. We've recently shipped Octopus Cloud alpha which means we took time to look at how to keep a healthy and secure Octopus. You can too!
visibility: private
published: 2018-02-27
metaImage: metaimage-freetrial.png
bannerImage: blogimage-freetrial.png
tags:
 - Security
---

Do you take care of your own Octopus? We do too! We still have our own Octopus Server, as a pet for deploying most of the things we build. Just like us, most of you probably have your own pet Octopus - but how safe and secure is your Octopus?

We recently shipped [Octopus Cloud](https://octopus.com/cloud) alpha where we learned, and re-learned, a bunch of lessons about making Octopus safe and secure. In this post I'll show you how to give your Octopus a security check up.

_Don't want to take care of your own Octopus any more? Perhaps you should sign up for [Octopus Cloud](https://octopus.com/cloud) today?_

Want the full picture? Here is our [in-depth guide to hardening Octopus](https://octopus.com/docs/administration/security/hardening-octopus).

## Safely Exposing Your Octopus Server

For Octopus Server to do useful things, you need to expose it to your users, your infrastructure, and possibly external services. If you are exposing your Octopus Server to the public internet or 3rd parties, you should absolutely use HTTPS over SSL. With built-in support for [Let's Encrypt](https://octopus.com/docs/administration/security/exposing-octopus/lets-encrypt-integration), there is absolutely no excuse to expose your Octopus Server over HTTP in the clear.

Learn about [safely exposing your Octopus Server](https://octopus.com/docs/administration/security/exposing-octopus).

Also, do you know Octopus works natively with [Proxy Servers](https://octopus.com/docs/infrastructure/windows-targets/proxy-support)?

## Safely Doing Work on Your Octopus Server

Your deployment process will normally deal with packages and execute scripts. Quite often those packages are pushed to a Tentacle or SSH deployment target, and your scripts will execute on those machines. However, many deployments don't need a Tentacle or SSH target - like deployments to cloud services or similar. In this case, it would be annoying if you had to set up a Tentacle or SSH target just to push a package to an API, or run a script but you don't care where that script runs.

In Octopus `3.0` we introduced the concept of a worker which can deal with packages and execute scripts without the need to install and configure a Tentacle or SSH target. By default Octopus Server using the [built-in worker](https://octopus.com/docs/administration/workers/built-in-worker) runs under the same security context as the Octopus Server. This is very convenient when you are getting up and running, but it isn't necessarily the best way to keep running long-term.

**We recommend every Octopus Server be configured to use the built-in worker as a different user, or use [external workers](https://octopus.com/docs/administration/workers/external-workers).**

Learn about [workers](https://octopus.com/docs/administration/workers).

## Hardening Your Host Operating System

How secure is the host operating system for your Octopus Server? If you aren't sure, [here are some tips](https://octopus.com/docs/administration/security/hardening-octopus#harden-your-host-operating-system)!

## Hardening Your Network

Do you allow unfettered network access in and out of your Octopus Server? Octopus actually uses a small number of well defined network protocols. We also provide some tips which will prevent attackers from using your Octopus Server as a vector into your network.

Learn about [hardening your network](https://octopus.com/docs/administration/security/hardening-octopus#harden-your-network).

## Upgrade

Are you running the latest version of Octopus Server? Generally speaking, the latest available version of Octopus Server will be the most secure. You should consider a strategy for keeping your Octopus Server updated. We follow a [responsible disclosure policy](https://octopus.com/docs/administration/security#disclosure-policy) so it is possible for you to be aware of any known issues which affect the security and integrity of your Octopus Server.

## Conclusion

Octopus is built with security as a first-class concern - but [you still have some work to do to keep your Octopus safe and secure](https://octopus.com/docs/administration/security/hardening-octopus). This blog post has a few hints and tips. If you want to give your Octopus a full checkup, use our [in-depth guide to hardening Octopus](https://octopus.com/docs/administration/security/hardening-octopus).

If you don't want to take care of your own Octopus, perhaps you should signing up for [Octopus Cloud](https://octopus.com/cloud) today?
