---
title: "Managing Spaces with Octopus Data Center Manager RFC"
description: We are designing a new product to manage Octopus Servers at scale. This is a request-for-comments.
author: shannon.lewis@octopus.com
visibility: private
tags:
 - RFC
---

In an [earlier post](octopuses.md) we talked about some of the real world problems our customers are having with Octopus at scale, and introduced some of our vision for solving those problems in Octopus 4.0. One of the things we left off with was imagining a tool to centrally manage a number of Octopus servers. In this post we're going to talk more about what we think that might look like.

## Octopus Data Center Manager
Our thinking is that this new tool will actually be a new product. Its working name is **Octopus Data Center Manager** (ODCM).

## Scenarios
We'll outline some specific usage scenarios in this section, and in the next we'll look at the ODCM features that will enable them.

### Scenario 1: Tina
Tina is responsible for a team who are delivering some new internal software. They are nearing their first delivery milestone and don't want any surprises. They have been working happily with a specific version of Octopus, they aren't experiencing any issues, and they want to keep it that way.

### Scenario 2: Bob
Bob is a developer who has some specialist skills and is going to do some work with Tina's team. He also works on another team who are using Octopus and he doesn't want new credentials or to have to remember a new Url.

### Scenario 3: Geoff
Geoff is a consultant from an external company who is also joining Tina's team. He already has an Azure AD login managed by his company, and would really prefer to not have another set of credentials to manage.

### Scenario 4: Malcolm
Malcolm is another team lead, who heads up a team responsible for some software that Tina's team will need to integrate with. He has a set of variables defined in his Octopus server that Tina's team will need access to to make there job easier. It's important that this information doesn't spread too far though, so he wants to be able to control who has access.

### Scenario 5: Barry
Barry is responsible for the company's Octopus infrastructure. Some critical projects rely on this infrastructure and it's important he knows when any Octopus servers go offline.

## Features
The following diagram shows what we think a typical ODCM installation might look like, and we'll refer back to it as we go through some of the features. ODCM is shown in a Highly Available style configuration (like Octopus Deploy itself, it will support single node and HA configuration depending on your requirements). A HA configuration will be important for Barry, to ensure he can always get reliable feedback on the rest of the Octopus servers.

![ODCM Architecture](odcm-arcitecture.png "width=500")

### Giving teams their own Space
When we started talking about ODCM and its functionality internally something became apparent pretty quickly that some of the terminology can be overloaded and/or confusing. This was within our team, and we're living this stuff every day, so how do we avoid confusing everyone else?

As an example, we started talking about splitting out Octopus _instances_. But we already use instance to mean a running [instance of Octopus Server or Tentacle EXE](https://octopus.com/docs/administration/managing-multiple-instances), and that's not what we meant. We thought about Octopus _servers_, as I used above, but server could be confused with machine.

What we were talking about is what's represented by the URL that the users use to access Octopus. Not physically, but conceptually. What we were talking about was the ability to split that so teams had their own Space in which to work. And so we started talking about Spaces.

ODCM therefore manages Spaces, and Barry will be able to use it to do things like:

- enlist existing "Octopus servers" as Spaces,
- separate things out of an existing Space into a new one,
- create a new blank Space
- monitor Spaces
- report across Spaces

### Identity management
One of the keys to working across multiple Spaces is to deal with user identity and access control. We can solve these problems by having ODCM take responsibility for them.

When a Space is enlisted with ODCM its authentication will be configured to point to ODCM, which will centralize identity management and allow SSO across Spaces.

This helps in a number of the scenarios:

- Bob can move team and doesn't need a new identity or need to find a new URL
- Lisa can locate Bob's existing identity to easily allow him access to her team's Space
- Lisa can create an external identity for Geoff, who can then log in with his existing credentials.

Once Bob has access to multiple Spaces, his user experience in Octopus will change slightly to allow him to switch Spaces quickly and effortlessly. We're thinking it'll look something like this.

![ODCM Space Switching](odcm-space-switch.png "width=500")

ODCM will support all of the external providers currently supported by Octopus Deploy. We are considering support federated authentication across organizations at some point in the future, but this isn't targetted for our initial release.

### Access control
We picture access control operating across Spaces at two levels:

1. who can access a Space?
1. what can a user do within a Space?

If you're Barry administering ODCM, you will be able to control which groups of users have access to which Spaces. A group may consist of Users and/or external groups (i.e. those sourced from Active Directory or Azure AD).

Similarly, if you're Lisa you'll be able to manage which groups of users have access to your Space and exactly what permissions they have in your Space. So for example, you could specify a Team which permits Developers to deploy things to the Dev environment. If Bob is a member of the Developer group in ODCM then when he is given access to the Space he'll be able to deploy to Dev immediately.

![ODCM Groups](odcm-groups.png "width=500")

### Sharing
When you only have a single Space, sharing of information is a non-issue. Once you have lots of smaller Spaces, there is an increased likelihood you'll want/need to share information between them.

Our vision for Spaces is that they should be collections of related things, so the need for sharing should be minimal. We thought about which things are likely to need sharing, and think they'll be things like:

- Step Templates
- Server Extensions
- Variables
- Tentacles

ODCM will most likely have the ability to host a version of the Community Step Template library, to share Step Templates between Spaces. We may also introduce a similar concept for Octopus Deploy server extensions, so they can be shared.

Sharing of Variable Sets is a little more complicated, because they could contain sensitive information, like in Malcolm's scenario. You will be able to specify which Variable Sets are shared and with which Spaces.

A Tentacle can already be used by more than one Octopus server, so this still applies and it can be used by more than one Space.

### Multiple Octopus Deploy versions
Let's now consider what happens when another team starts up and Barry gets a request for a new Space. He looks across the current machines and sees that one has capacity to host another Space. It happens to be the machine that's hosting Lisa's Space. So whilst the machine has capacity, the Octopus version being used by Lisa's Space must not change.

The current Octopus Deploy MSI installer creates a problem here because it only allows a single version to be installed on a machine, by virtue of "*C:\Program Files*". You can use Octopus Server Manager to configure multiple instances on a single machine, but they are all sharing the same binaries and are therefore the same version.

You can work around this today but it takes some effort. We want to make it easy. Our current idea is that we'll include an agent on the machines hosting Spaces and automate the deployment of Octopus server itself.

Once there are multiple Spaces, and even versions, of Octopus Deploy server running on a machine, maintaining isolation will be important. We haven’t settled on exactly how we'll ensure this, but expect there will be some changes required to the way Server Side scripts operate.

### Octopus Deploy Server monitoring and reporting
If you're Barry and you are managing a single installation of Octopus, getting consolidated information across projects/teams is relatively easy. Once you have lots of installations, it's not easy.

ODCM is well positioned to help with this. It will be in communication with the Spaces, so it can request information and statistics that can be collated centrally. A dashboard and some reports will provide access to this information.

We don't expect that all of the dashboard and reports will make it into the initial release. We will focus on a minimal set and build on this over subsequent releases. The initial release may contain something like:

- a dashboard showing Spaces, with their server version and current status (online/offline)
- a report showing project count and target count per Space
- a report showing deployment counts (number of successful and failed deployments) per Space over a give timeframe

If there are other metrics you think would be valuable, please let us know.

## Licensing
ODCM will be a separate product to Octopus Deploy itself, and will have a different licence model. The model is still under discussion and we’ll share more details as they become available. The pricing will be friendly in regards to running lots of Spaces.

## Feedback
What we've talked about above is what we think will be the minimum viable product for ODCM. As always, we're keen to get your feedback and from there we'll be looking to start the implementation in the next couple of weeks.
