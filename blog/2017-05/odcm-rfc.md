---
title: "Managing Spaces with Octopus Data Center Manager RFC"
description: We are designing a new product to manage Octopus Servers at scale. This is a request-for-comments.  
author: shannon.lewis@octopus.com
visibility: private
tags:
 - RFC 
---

When we first built Octopus, we imagined it would be used by small teams to deploy applications to a dozen or so machines. Over time, we've [seen customers scale Octopus up to many thousands of machines](https://octopus.com/blog/octostats), deploying hundreds of different projects. At that scale, customers need their Octopus instances to be online at all times, so we support running a single [Octopus server across a multi-node, high availability cluster](https://octopus.com/high-availability).

One great big Octopus server isn't always a great idea though, especially when it's used by a large number of teams that don't share a lot in common. That was the case at Accenture, who [standardized on Octopus](https://channel9.msdn.com/Shows/ANZMVP/Updating-Octopus-Deploy-at-Accenture-with-Jim-Szubryt-and-Damian-Brady) across the organization, and had many hundreds of teams on a handful of very large Octopus servers. For their scenario, it made much more sense to split the big Octopus servers into lots of small ones, effectively giving each team or handful of teams their own small, isolated Octopus servers.

When it comes to rolling out Octopus at scale, we want to offer more choices:

1. One or two really big, highly available instances with great performance
2. Lots of teeny, tiny Octopus instances (which might also be highly available)
3. Some mix of the above

While we've had a great story for a while now for scenario #1, we haven't had great solutions for #2 and #3. In this post, we're going to outline a new product offering that we're working on for enterprise-wide adoption of Octopus, that's designed to make all of these scenarios easier. 

## The case for lots of small instances
A large, single HA Octopus makes sense when you have a team that share a lot of projects and machines - imagine something like a team building an ecommerce platform that runs across hundreds of machines, deploying often and rolling back if something goes wrong. The existing Octopus HA solution is perfect for this.

Where it doesn't make sense, is when the Octopus server is used by many disparate teams or departments, without a whole lot in common. We'll continue to do work to make Octopus easier to navigate when you have lots of unrelated projects and environments, but sometimes it would just be better if every team got their own Octopus instance.

Here are some problems to having one big Octopus instance when so many different teams are involved:

- Maintaining permissions for each team on specific projects is tedious
- It is not possible to restrict many concepts to specific teams, e.g. NuGet feeds, variable sets and machine policies
- Large numbers of projects, environments and tenants cause queries to run slowly, e.g. Dashboard
- Deployments for other teams that the user may have no control over (or access to) impact the deployment of their projects
- Upgrading an Octopus instance impacts everyone. This causes a tension between leading-edge teams that want the latest Octopus features, and more stable teams that might be in a freeze or undergoing an audit. 
- It’s not possible to delegate permissions - e.g., the manager of a dev team can’t add a new developer to a team, they have to ask an Octopus administrator to do it.
- Backup and restore of an individual team is practically impossible. What do you do if one team makes a big mistake and needs a backup restored, while other teams have moved forward? How does this impact on your compliance obligations?
For these reasons, we've been recommending that customers run lots of small instances, perhaps one for each team or each department.

**You can already do this today.** We have customers, like Accenture in the video linked above, but also others, who are doing this in production at scale with lots of small Octopus instances.

- You can share the same SQL Server cluster, but give each team their own SQL database
- You can run many instances of Octopus (each pointed at a different database) on a single machine, or in high availability mode across machines

The best part about this is that you gain so much more ability to scale - you can effectively "shard" your Octopus across many machines and many database servers.

## Points of friction

While it can be done today, the major downside is that we don't currently provide any tooling to help mange all those little Octopus servers - the experience for end users and Octopus administrators just isn't great.

- If the instances are installed on the same VM, they all have to be upgraded together
- Users need to remember/bookmark all the URLs for the servers they have access to
- Users need to log on to each instance
- When a new user joins, they need to be added to teams on every instance

## Addressing the friction
To make it easier to more customers move towards this model of having lots of smaller instances, we want to address these friction points. To do that, we want to make the following easier:

- For administrators to manage multiple smaller instances of Octopus Deploy, including configuration and upgrades
- For administrators to manage users and groups across those instances of Octopus Deploy
- For teams to become more self-managed and independent - so they can manage their own Octopus Projects, Environments, Machines, Variables, Step Templates etc without worrying about the impact on other teams
- For users to switch between all the instances they have access to using Single Sign-On (SSO)
- For administrators and managers to be able to view reports and information across all of the instances

## Octopus Data Center Manager
When we looked at the friction points outlined above, they're all a direct result of working across Octopus Deploy installation boundaries. The answer we're looking for seems to be a layer above Octopus Deploy itself, and so we're  building a new product called **Octopus Data Center Manager** (ODCM).

Let's have a look through what we see as some of the key features for ODCM, and look at how they address the points above.

### Spaces
When you first start using Octopus Deploy you will likely start with a standard installation and over time it grows. As the installation grows, you will reach a point where your Octopus feels cluttered. When you reach that point you will also probably be able to identify collections of related things that could be grouped together and separated from everything else. What you would like to do is move those into their own Space, i.e. their own smaller Octopus Deploy.

We can help you manage this using ODCM. It allows you to:

- enlist existing installations as Spaces,
- separate things out of an existing Space into a new one,
- create a new blank Space

### Identity management
One of the keys to working with multiple instances of Octopus is to deal with user identity and access control. We can solve these problems by having ODCM take responsibility for them.

When you enlist a Space with ODCM we will configure the Space so authentication is handled by ODCM. This means you can centralize identity management and SSO.

Now if you want all your users to sign on with Active Directory, Azure AD, Google Apps, or any other supported identity provider, you would configure this in one place: your ODCM.

### Access control
When you have multiple Spaces, access control operates at two levels:

1. who can access a Space?
1. what can a user do within a Space?

If you're responsible for administering ODCM, you can control which groups of users have access to which Spaces. A group may consist of Users and/or external groups (i.e. those sourced from Active Directory or Azure AD).

If you're responsible for a Space, you can add groups from ODCM to a Team control permissions.

<img src="https://i.octopus.com/blog/201704-odcm-groups-947H.png" style="width:500px"/>

### Switching Spaces
The features so far have focused on the management side of having multiple instances. Let's talk now about one of the friction points for the end users, remembering where all of the Octopus installations are that they have access to and signing in to each one separately.

Each user currently has to remember/bookmark the list of server Urls for themselves. This is cumbersome and makes discovery difficult, and we want to address both of those.

Within a Space, we are aiming to keep the changes to Octopus Deploy itself to a minimum. One change we do see is a UI update to help the user change Spaces rapidly.

We're thinking that something similar to the following may be what the user will see.

<img src="https://i.octopus.com/blog/201704-space-switch-6HZ4.png" style="width:500px"/>

The new control would appear if the user has access to more than one Space and would might have a user experience something like when you're changing boards in Trello.

Single Sign On (SSO) will be enabled across the Spaces, so switching will not require the user to log in again.

### Sharing
When you only have a single installation of Octopus, sharing of information is a none issue. Once you have lots of smaller installations there is an increased likelihood you'll want/need to share information between them.

In our current vision for Spaces, they should be a collection of related things and so the need for information to cross Space boundaries should be minimal. The things we think are most likely to need sharing are:

- Step Templates
- Server Extensions
- Variables

To share Step Templates, we think ODCM will have the ability to host a version of the Community Step Template library and the Spaces can then be configured to point to it instead of the current public library.

Library Variable Sets will continue to provide sharing of variables within a (Space). We think they'll also be what is used to share variables between Spaces, by allow a Project to reference a set from another Space.

We are still assessing the implications of permissions and access to sensitive variables when crossing Space boundaries.  Our initial thoughts are that for a given Space there will be a way to configure which other Spaces are allowed to access its Library Variable Sets.  This will give the variable set owner control over who can use the variables.

### Multiple Octopus Deploy versions on a single machine
On to sharing of a different kind. By virtue of "c:\Program Files" the current Octopus Deploy MSI installer only allows a single version to be installed on a machine. Using Octopus Server Manager you can configure multiple instances on a single machine, but they are all sharing the same binaries and therefore must all be updated at once.

If Teams are sharing hardware then this is counter to the objective of allowing them to become more self-managed and independent, so we'll be looking to update the installation process.

For a standard installation the process won't change, what we are thinking is that we'll augment it by including an agent on the machines that will host Spaces and ODCM will be able to use that to deploy and manage Space instances on the machine. In other words, ODCM and the agent will use the same model as Octopus Deploy server and Tentacle, and we'll be able to deploy Octopus instances in essentially the same way Octopus deploys applications.

The Team can then control version updates on their Space via ODCM.

Again, we aren't sure if the following will make it into the initial release, but from here the agent could also be extended to include functionality like:

- Automatically install new releases to Spaces (if the Space has opted in and enabled this)
- Ensure nodes in a HA Space are all running the same version and manage updating all nodes to a specific new version when required

### Server Side scripts
Once there are multiple Spaces, and even versions, of Octopus Deploy server running on a machine it will be imperative to maintain isolation between Spaces.  As part of achieving this, the way server side scripts are executed will likely change to some extent. We haven’t settled on exactly how this will operate, but we’ll be aiming for the smallest change to the user experience/setup as possible.

### Tentacles
A Tentacle can already be used by more than one Octopus server, so this applies in the new model too. I.e. it can be used by more than one Space.

### Licensing
ODCM will be a separate product to Octopus Deploy itself, and will have a different licence model. The model is still under discussion and we’ll share more details as they become available, the pricing will be friendly in regards to running lots of Spaces.

### Octopus Deploy Server monitoring and reporting
Similarly to what we talked about earlier with information sharing, as someone managing a single installation of Octopus getting consolidated information across projects/teams is a relatively none issue. Once you have lots of installations, it's cumbersome.

ODCM will provide centralized reporting across instances, for example, how many deployments has the whole organization done this week/month.

ODCM will also provide a global dashboard for users to see what’s been deployed across the many spaces they care about.

We don't see all of the dashboard, reports etc being essential for the initial release. We will focus on some core reporting and build on this over subsequent releases.

### ODCM installation topology
To help visualize what an ODCM installation might look like, the following diagram shows a scenario where there are two server instances (A and B) that have been registered as Spaces.  ODCM is shown on the right of the diagram, and is illustrated in a HA style configuration (like Octopus Deploy itself, it will support single node and HA configuration depending on your requirements).

<img src="https://i.octopus.com/blog/201704-spaces-arch-Z87V.png" style="width:500px"/>

## Feedback
What we've talked about above is what we think will be the minimum viable product for releasing ODCM, save for the couple of caveats mentioned on the way through.

As always we're keen to get your feedback, and from there we'll be looking to start the implementation in the next couple of weeks.

