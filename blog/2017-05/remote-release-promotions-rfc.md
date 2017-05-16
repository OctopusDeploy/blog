---
title: "Remote Release Promotions RFC"
description: We are designing a new feature to allow promoting Releases between different Octopus Servers (Spaces). You may want to do this for for security, geographic, or other reasons. This is a request-for-comments.  
author: michael.richardson@octopus.com
visibility: private
tags:
 - RFC 
---

## The problem

There are scenarios where it makes sense for different Octopus Server instances to perform deployments depending on which environment is being deployed to. For example:

- Octopus Server 1 deploys to Development and Testing environments
- Octopus Server 2 deploys to Staging and Production environments

### Elevator pitch

We are planning a feature which enables you to promote releases across multiple Octopus Servers... in a nice way. :) If you are trying to do this today, you know it hurts real good.

*TODO: INSERT PRETTY PICTURE HERE*

The two most common reasons for this are:

- Secure environments
- Geographically distant environments

### Secure environments

For security purposes many organizations separate their production and development environments. A common driver for this is achieving PCI DSS compliance.

The secure zone may even be completely disconnected (aka air-gap).

These organizations still want all of the Octopus-goodness, like promoting the same release through the environments, and seeing the progression on the dashboard. But they don't want the development Octopus Server to be connected to the production environment. It's also likely likely want a different set of users (possibly from a distinct Active Directory domain) to be have permissions to the production Octopus Server.

*TODO: INSERT PRETTY PICTURE*

### Geographically distant environments

Other organizations may deploy to geographically-distant environments.

For example, their development environment may be located in Brisbane (it's a great place to live!), while their production environment is hosted at data centers in the US and Europe.

The problem with this currently, is that the packages are transferred at deployment time. These customers would like to promote the release at a time of their choosing (including transferring the packages), then perform the deployment with the packages already located in the appropriate data center, close to the target machines.

## Proposed solution

Our proposed solution will enable you to spread your entire deployment lifecycle across multiple "Spaces". A "Space" is a concept we are [planning to introduce](/blog/2017-05/odcm-rfc.md), where each "Space" has its own set of projects, environments, lifecycles, teams, permissions, etc.

Imagine if you could add a Space to your Lifecycle, just like you can add environments, and then promote a release to another Space. When you promote a release to another Space, Octopus could bundle up everything required to deploy that release into the environments in the **other** Space. We will also cater for scenarios where there is strict separation between your Spaces (think PCI DSS). That's why we're currently calling this feature **Remote Release Promotions**.

- High-level architecture diagram (pretty picture) (Vanessa)
- Release Lifecycle (showing promotion through environments, then zones) (Vanessa)

We think there are two major concepts at play here: **Lifecycles** and **Trusts**.

### Lifecycles

We think Lifecycles should be **defined** within a Space and able to be **composed** across multiple Spaces - you can think of it like chaining Lifecycles from different Spaces together.

**Define within a Space:** This gives the teams in each Space the ability to manage their own environments and Lifecycles how they see fit. For example, a member of one Space might decide to introduce an environment into their Lifecycle. We don't want the decision to introduce an environment into a Lifecycle in one Space to have any impact on any other Spaces.

**Compose across Spaces:** This gives you the ability to model your overall deployment pipeline as a **Composite Lifecycle** made by connecting together Lifecycles which are defined in different Spaces. For example:

1. You might want to promote a release through your test environments, then promote the release to one or more Spaces that manage the production environments.
1. You might want to promote a release through your Dev team's test environments, then promote the release to another Space managed by a QA team. When they are finished testing you want the Dev team to promote that same release to yet another Space where the Operations team manages your production environments.
1. You might want to do the same as #2, but once the QA team is finished they promote the release directly to the Operations team's Space without going back through the Dev team.

### Trusting other Spaces

We already have the concept of establishing trust between [Octopus Server and Tentacle](https://octopus.com/docs/reference/octopus-tentacle-communication): it will only execute commands sent from a trusted Octopus Server. We also think it's important that a trust relationship is established between two Spaces before they start sharing things like "everything required to deploy a release" and "the results of deploying a release". We talked about [sharing](/blog/2017-05/odcm-rfc.md#sharing) in our recent blog post introducing the concept of spaces and the Octopus Data Center Manager (ODCM).

At its core this relationship will consist of a **Name** and an **X.509 Public Key Certificate**. This will enable each Space to uniquely identify the source of information, and validate the integrity of the information, just like [Octopus Server and Tentacle do today](https://octopus.com/docs/reference/octopus-tentacle-communication). We think the best way to configure this relationship is using ODCM since its core capability is managing Spaces.

This means you are in control of which information flows between different Spaces, and you can audit it all in one place.

## Definitions

In the rest of this RFC we are going to introduce some new terms. Let's define them here so we don't all get horribly confused.

- **Space:** Contains a set of projects, environments, variables, teams, permissions, etc, bounded by a single Octopus database. Learn more in our recent [RFC](/blog/2017-05/odcm-rfc.md).
- **Release Bundle:** A package containing everything required to deploy a specific release of a project.
- **Deployment Receipt:** A document containing everything required to show the result of deploying a specific release of a project.
- **Source Space:** The Space that owns the project and its releases, and where release bundles are created if you decide to cross Space boundaries.
- **Target Space:** The Space where a release bundle will be imported. The release extracted from the release bundle can then be deployed to environments in this Space.
- **Remote Environment:** A reference to an environment owned by another Space.
- **Remote Project:** A reference to a project owned by another Space.
- **Remote Space:** A Space managed by a different ODCM, usually in a different secure network zone. The concept of a **Remote Space** will enable you to promote releases across secure network boundaries.
- **Variable Template:** We introduced this concept with multi-tenant deployments. In this context you could express that a variable value is required for each environment a project can be deployed into.

## Example: Secure Environments

Let's explore this concept using the **Secure Environments** example we mentioned earlier, where you want strict separation between your development and production environments. In this case we will model this separation using two Spaces:

- `DevTest Space`: where your application is deployed for development and testing purposes
- `Prod Space`: where the production deployments of your application will be deployed and strict compliance controls are required

_IMAGE: Show two Spaces, indicating where project, and each environment is owned, and how the bundle flows_

Let's consider how each different person in your organization might interact with Octopus to promote a release across these two Spaces all the way to production.

### Configuring Spaces (Mike N)

A good place to start is by configuring your Spaces and establishing a trust relationship between them. In cases like the Secure Environments scenario, we think you will end up installing an instance of ODCM inside each secure network zone. This will allow your teams to independently manage the Spaces inside each zone, and configure trusts between Spaces in the same zone or across different zones as required.

We think the overall process will look something like this:

1. Configure your secure network zones
1. Configure an instance of ODCM in each zone for managing the Space in that zone
1. Use ODCM in the **production zone** to create the `Prod Space`
1. Use ODCM in the **development zone** to create the `DevTest Space`
1. Use ODCM in each zone to configure a trust relationship between your Spaces

Since there is strict separation between the zones you will have to configure two **Remote Spaces** to trust each other manually, by exchanging the X.509 Public Key Certificates for each Space:

1. Use ODCM in your **development zone** to download the X.509 Public Key Certificate for the `DevTest Space`.
1. Go to the ODCM in your **production zone** and create a new **Remote Space** called `DevTest Space` giving it the X.509 Public Key Certificate you downloaded for the `DevTest Space`.
1. Use ODCM in your **production zone** and download the X.509 Public Key Certificate for the `Prod Space`.
1. Go to the ODCM in your **development zone**, create a new **Remote Space** called `Prod Space` giving it the X.509 Public Key Certificate you downloaded for the `Prod Space`.

Now the `DevTest Space` knows about the existence of the `Prod Space` you will be able to promote releases to that Space. Additionally, since you've exchanged public keys, the `Prod Space` can trust Release Bundles promoted from the `DevTest Space`, and `DevTest Space` can trust Deployment Receipts from the `Prod Space`!

### Working with projects

We don't see very much changing - life will pretty much go on just like before. You will still be able to change the deployment process, manage variables, and create and deploy releases to environments in the `DevTest Space` just like normal. However in this example the `Production` environment is owned by the `Prod Space`, meaning the `DevTest Space` has no concept of this environment:

- How do you provide variable values that will be used when deploying to the `Production` environment?
- How do you configure special steps of your deployment process so they only execute when deploying to the `Production` environment?
- How do you show the result of deployments to the `Production` environment on your dashboards?

Please welcome **Variable Templates** and **Remote Environments**!

#### Variable Templates

Imagine if you are the person importing a release bundle into your Space - how do you know which variables need values for each environment in your Space? And even if you know which variables you need to set, what should you set the value to?

Now imagine as a project contributor if you could express that a variable value is required for each environment a project can be deployed into. And imagine you could define a data type for the variable, provide help text, decide whether the value is mandatory or optional, or even provide a default value.

Variable templates could make it much easier for a person importing a release bundle into their Space to "fill in the blanks".

:::hint
This would also be really handy even if you are only promoting releases within your own Space. Using variable templates, if you introduce a new environment into your own Space, Octopus will prompt you for those variable values.
:::

We introduced the concept of [Variable Templates](https://octopus.com/docs/deploying-applications/variables/variable-templates) for multi-tenant deployments in Octopus 3.4. We would like to build on this concept further as part of this set of features.

_IMAGE: Variable Template Editor?_

Learn more: _LINK: Variable Template GitHub Issue_

#### Remote Environments

We want to enable scenarios where you promote releases to other Spaces without needing to know anything about the environments in that Space. However, we can see scenarios where you will want to know about environments in other Spaces:

- you want certain steps to be executed when deploying a release to the `Production` environment
- you already know a handful of variable values required when deploying a release to the `Production` environment (perhaps they aren't secret)
- you want to see the results of deploying a release to the `Production` environment on your own dashboard

The problem here is that the `Production` environment is owned by the `Prod Space`, so your `DevTest Space` doesn't know the `Production` environment exists! Imagine if you could add a **Remote Environment** to the `DevTest Space`. This remote environment would be like a placeholder for the real `Production` environment. Octopus could even name it `Prod Space: Production` so we are all clear about the ownership of this environment. _Think of this like namespaces: so you can have a `Production` environment in multiple Spaces._

We think the process would be something like this:

1. Go to the Environments page and click the `Add environment` button
1. Octopus could show a list of Spaces it knows about, in this case the `Prod Space`
1. Select `Prod Space` as the owner of the environment (indicating this is a remote environment)
1. Name the environment `Production` so it matches

Now that you have configured the `Prod Space: Production` environment:

- you could scope steps to `Prod Space: Production`, and those steps will be run when a release is eventually deployed to that environment.
- you could set variable values in your `DevTest Space`, scope them to `Prod Space: Production`, and they will be used when a release is eventually deployed to that environment.
- Octopus could show the `Prod Space: Production` environment on the dashboard in the `DevTest Space`.

### Configuring a Lifecycle including other Spaces

In order to promote a release to the `Production` environment, you will need to configure a Lifecycle with the ability to target the `Prod Space`. We think you should be able to add Spaces into the Phases of your Lifecycle just like you can add environments in Octopus today. This would work quite nicely for our example scenario where you just want the release promoted to the `Prod Space`.

What if you wanted to create a more complex Lifecycle? For example, you promote releases to a `QA Space` for testing by the QA team, and wait for them to finish testing it before Octopus will allow you to promote that release to the `Prod Space`? We think you should be able to add **Remote Environments** to your Lifecycles making Octopus behave just like that environment was part of the same Space.

### Promoting releases to other Spaces (Mike N)

Eventually you want to deploy a release to the `Production` environment! Since you have added the `Prod Space` to your Lifecycle, you could promote your release to the `Prod Space`. At this point Octopus would create what we are calling a **Release Bundle**: a set of files including everything required to deploy that release to environments owned by other Spaces.

In our example somebody would have to manually transfer the **Release Bundle** to the `Prod Space` and import it. If your Spaces are able to be connected, Octopus could automate a lot of this process for you.

#### Release bundles

Up to this point we've talked about a Release Bundle but we haven't gone into too much detail. This is some of our thinking and certainly an area where we would like your feedback:

1. You could click a button in the **Source Space** called something like `Promote 3.2.5 to Prod Space` to start the process
1. The **Source Space** will build a Release Bundle specifically crafted to transport the required information the **Target Space**.
1. The Release Bundle will not include the packages themselves, but instead it will include a manifest of all the packages required by the release, including the ID, Version, and Hash:

    - This will enable the packages to be transferred or replicated to the other Spaces in the most efficient manner possible, perhaps using delta compression, or you might want to take care of this yourself.
    - This will also enable the **Target Space** to validate the identity and integrity of the packages being deployed - they are guaranteed to be the same ones that were tested.

1. The Release Bundle will include essential details of the **Source Project**, the release, the deployment process snapshot, and the project variable snapshot:

    - Any variable values and parts of the deployment process that would never apply to the **Target Space** will be omitted from the bundle
    - Any variable values and parts of the deployment process that would never apply for releases in this channel

1. The Release Bundle will include summary details of the deployments up to this point in time so they can be optionally displayed on the dashboard in the **Target Space**.
1. When building the Release Bundle the **Source Space** will encrypt any sensitive information with the X.509 Public Key Certificate of the **Target Space** so it can only be decrypted by the **Target Space**.
1. When building the Release Bundle the **Source Space** will digitally sign a manifest with the X.509 Private Key Certificate of the **Source Space** so the **Target Space** can validate the source and integrity of the bundle before importing it.

### Importing releases into your Space (Mike N)

Once the **Release Bundle** has been transferred to the `Prod Space` you will need to import it. There will be a lot of details to figure out, but at the highest level we expect the process to look something like this:

1. You could be shown a list of **Release Bundles** ready to be imported and you choose to import one.
1. You could be shown a display including the packages required for the release, bundled variable values, variable templates, and the deployment process.
1. The project will be imported as a **Remote Project**. Similar to a **Remote Environment** your project would be namespaced like `DevTest Space: My Project`. We also think the **Remote Project** should be largely read-only, and will probably use a fairly different UI to normal projects.
1. The release itself will be imported along with the deployment process snapshot and variable snapshot that were frozen when the release was created.
1. You will need to choose the Lifecycle you want to use for promoting this release through the environments in the `Prod Space`.
1. Octopus will prompt you to set any missing variable values for your environments and tenants before the release can be deployed.

:::hint
The person importing **Release Bundles** will need to be granted all the permissions to create and edit projects, variables, packages, etc.
:::

#### Mostly read-only

We think it's worth calling out: almost everything that will be imported will be read-only, and some concepts won't even be transferred across Space boundaries. The end goal is to reliably deploy a release into your environments avoiding as much manual intervention as possible. There are still a lot of details to sort out, but we think a good rule of thumb will be:

- Anything used to build a release will be read-only in remote Spaces
- Anything used to customize a deployment will be editable in remote Spaces

For example, we expect you will want the deployment to use the process as it was when the release was created (repeatability) but have the chance to set the correct database connection string for your environments/tenants (variability). Here are some other examples:

- **Project Triggers** are all about automatically triggering deployments - they should be configured where the deployments are happening
- **Tenants** are about allocating deployment targets and defining deployment variable values - they should be configured where the deployments are happening
- **Channels** only really matter up to the point where you create a release, and you will need to choose a Lifecycle when importing - channels should be configured where the releases are created

#### Deployment process and variable snapshots

We think an important part of this feature will be the ability to view and understand the deployment process and project variables that were frozen into a snapshot when the release was created. Imagine trying to import and approve a release for deployment without being able to see the process and variable values that will be used during deployment?

This is actually a problem we've wanted to solve for quite some time: in Octopus today, you can see the variable snapshot (if you can find the correct link) but you cannot see the deployment process as it was defined when the release was created. Imagine if you could even view releases side-by-side to compare them with each other!

### Approving a release (Mike N)

- Idea for multi-team sign off on a release before it is allowed to be deployed - no need for manual intervention steps for this purpose

### Deploying releases (Mike N)

Now the release has been accepted it can be deployed to the environments in the `Prod Space`. For all intents and purposes this would work just like the release was created in the `Prod Space`: all the same rules would apply for deploying this release including:

- Project permissions: teams could be restricted to **Remote Projects** just like normal projects - after all, they are just normal projects but owned by another Space
- Environment permissions: teams in the `Prod Space` could be granted appropriate permissions to environments in the `Prod Space`, just like normal
- Lifecycle progression: Octopus will ensure each release progresses through the appropriate Lifecycle in the `Prod Space`, just like normal

### Aggregated dashboard (Mike N)

- Wants to see an aggregated overview of the deployments for the entire lifecycle

## Release bundle (Michael R)

- What goes across?
- Packages (hash)

## Release Acceptance (Michael R)

- Review and accept diffs?
- Provide variable values

## Flowing deployment results back across (Michael R)

- Deployment receipts
- Updating the source dashboard

## Variables (Michael R)

- Environment Variable Templates

## Super nitty gritty (Michael R)

- Disconnected mode - details on how we see this working.
- Version tolerance/message schema
- Snapshots (how to update on the source Space and re-promote to remote Space)
- What will be locked on the target Space?
- We want to use delta compression
- Tenants could span Spaces
- Channels in remote Spaces
- ARC in remote Spaces

## Security Concerns

- Two-way trust using PKI (like Octopus and Tentacle)

## Superseded Solutions (Vanessa)

- Octopus Migrator
- Offline Drops
- Octo.exe export/import
- Custom scripting using the Octopus REST API
- Manually migrating everything

## Rollout

- Phases
- Octopus v4?

## Feedback
