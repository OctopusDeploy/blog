---
title: Octopus Deploy 4.0 - Configuration
description: Server configuration is getting a whole lot easier, and more visual.
author: shannon.lewis@octopus.com
visibility: private
tags:
 - New Releases
 - Configuration
---

This post is a part of our Octopus 4.0 blog series.  Follow it on our [blog](https://octopus.com/blog) or our [twitter](https://twitter.com/octopusdeploy) feed.

---

For most Octopus users, their day to day interaction with Octopus involves designing and defining awesome deployment processes and then monitoring deployments based on those processes. They live in the beautiful world of the Octopus portal, which as you've already seen in this series is getting even more beautiful in v4.0.

There are another smaller group of users who live in a very different world. They are the group responsible for provisioning the Octopus server itself and keeping it humming along happily such that it can provide the beautiful world in which all of the other users live.

The world this group of users lives in today looks a bit more like this

```bash
Octopus.Server.exe stop --instance=master
Octopus.Server.exe configure --instance=master --corsWhitelist=http://opsthing.mycompany
Octopus.Server.exe start --instance=master
```

Now we know that world is beautiful to some, but it can present a couple of problems.

# Understanding the problems

## There are how many configuration options?

For those who've never had the pleasure of using the `configure` command, take a minute to either run `Octopus.Server.exe configure --help` (you may want to bump up the height of your Screen Buffer before running the command ;) ) or check out our [documentation page](https://g.octopushq.com/ConfigureCommand).

Understanding which of those options relate to each other is sometimes decipherable through the naming, but not always. Wading through all of those options can be a harrowing and frustrating experience.

## Console

So to change the Octopus server configuration you need access to the server's console. This usually involves physical access to the server, Remote Desktop or Remote PowerShell or something similar that will let you run commands remotely.

But what about API first?

## Desired State Configuration

What if you're trying to automate the deployment of Octopus server? You can use something like the following to describe the state you want the Azure AD configuration to be, for example

```bash
Octopus.Server.exe configure --instance=master --azureADIsEnabled=true --azureADIssuer=https://login.microsoftonline.com/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx --azureADClientId=zzzzzzzz-zzzz-zzzz-zzzz-zzzzzzzzzzzz

```

But it feels a bit cumbersome.

What about if you wanted to do something like drift detection? Well in v3.5 we introduced the [`show-configuration` command](https://g.octopushq.com/ShowConfiguration) which can help with that. It was actually originally intended as a support tool. It provided a quick way customers could get their entire configuration if they needed it when working with support. As it progressed we realized that by including a JSON format it could be really useful when doing drift detection.

This was a great step forward, but you can read the configuration in this nice JSON format that's easy to work with then when you find a value that has drifted you need to generate a call to the command line to make the correction, which again feels cumbersome. In other words, the reads and writes are very different shapes and are clumsy to work between.

## Extensions

In the section above the Azure AD configuration options were used as illustration. The interesting part about those configuration settings is that they don't come from the core Octopus server, we have an extension point and they are provided by the Azure AD Authentication Provider extension.

The nature of the way the configure command had originally been written made this an easy extension point to provide. The command is based around a list of options, and we just provided a way for the extensions to contribute to that list.

We started thinking about what it would take to allow access to the configuration settings in the portal. Due to the nature of the Angular app at the time, providing a read only view was all we could easily manage. If you haven't seen the configuration settings, it's available via a **Server settings** button on the Configuration -> Nodes page.

# The solution

We've had a number of these problems on our mind for a while now, and allowing editing of the configuration through an API seems like the obvious answer. So in 4.0 that's what we're doing.

## API first

Allowing reading and writing of the configuration through an API is core to making all of the problems we discussed above easier to solve. It means that we can surface the values in a more meaningful way in the UI and also allow editing like any other resource (more on that below).

Configuration via API also means that all of the data for reads and writes is exactly the same shape. If you're doing drift detection you can call the GET, look at the values in the document, change any that have drifted, POST it back and you're done. If you're setting up a new server, just POST the JSON and you're done.

## Getting things together

Earlier we mentioned that the settings are treated as a list, and working out which things in the list relate to each other can be difficult. Having an API helps us address this too. 

Given it uses a JSON document, it can use objects to represent configuration sections and related values. For example, Web Portal configuration can be represented something like this

```json
{
  "Id": "webportal",
  "Security": {
    "CorsWhitelist": "http://localhost:9005",
    "ReferrerPolicy": "no-referrer",
    "ContentSecurityPolicyEnabled": true,
    "HttpStrictTransportSecurityEnabled": false,
    "HttpStrictTransportSecurityMaxAge": 3600,
    "XOptions": {
      "XFrameOptionAllowFrom": ""
    }
  }
}
```

## A new UI

This was a big one. With the updates that are being made to the portal, it is now considerably easier for us to dynamically generate the forms for editing the configuration.

Why's that important? In a word, Extensions. We wanted a way to allow the extensions to contribute UI to the portal without it having to understand the technology involved. In v3.5+, the extensions were able to contribute Angular modules and this worked for what we needed to support the authentication provider extensions, but always felt risky.

So we're moving away from that model and instead using a model based on how we handle Step Templates. I.e. we describe the UI we want using some metadata and then generate it. This means there are actually 2 new APIs, one for getting/setting the values and one for getting the metadata related to the values.

#  The fine print

There are some caveats to these configuration changes and all relate to when you're using Octopus in a HA configuration.

## Node specifics

The only configuration settings you can see and edit via the API and UI are those that relate to every node in a HA node set.

Settings like ListenPrefixes, ForceSSL and RequestLoggingEnabled are node specific, for a variety of reasons, and therefore cannot be edited via the API.

## Performance and caches

One of the reasons those node specific settings are problematic is that changing them requires a restart of the Octopus service. In a single node configuration this isn't too hard to manage, but in a load balanced HA configuration it's a much trickier proposition and tackling that is beyond the scope of what we are aiming for at the moment.

So the settings we're going to allow editing of are stored centrally and applicable to all nodes. What could go wrong? Yeah, ok the section title gave it away. A number of the configuration settings, especially the Web Portal ones mentioned earlier, are cached on each node for performance. Now we have a distributed cache problem.

At this point, I'll point out that in v3.x that cache is loaded at server start and any changes to these values require a server restart. The goal in making this change was to both allow the values to be edited and update the cache without requiring a restart when they are edited.

Ok, so we have a distributed cache with no timeout. Step 1, make the cache expire. We looked at a couple of options around this, but settled on utilizing the server's heartbeat mechanism as the simplest answer. Every time the server heartbeats, reset the cache. The caveat is therefore that changes take effect without requiring a restart but may take a few seconds to actually come into effect.

Two final points on this, on the node that receives the API request we do actually reset the cache instantly. In a non-HA installation, where there is by definition only 1 node, this instance reset will be the behavior you get.

Lastly, if you do edit the configuration using the `configure` command line, you aren't in the same process as the actual Octopus service, so the cache can't be instantly reset and you have to wait for the next heartbeat. You will see a warning appear on the command line to let you know when you've changed one of these values and have to wait for the cache reset. Best way around this is to configure via the new API ;)



# Feedback welcome

As always, we're keen for your feedback so please leave comments below.



Happy Deployments! 