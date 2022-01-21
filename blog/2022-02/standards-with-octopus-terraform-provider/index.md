---
title: Use Octopus Terraform Provider to create standards
description: Learn how to leverage the Octopus Deploy Terraform Provider to create standards for all spaces on your instance.
author: bob.walker@octopus.com
visibility: public
published: 2022-12-31-1600
bannerImage: 
metaImage: 
bannerImageAlt: 
isFeatured: false
tags:
- DevOps
- Terraform
---

The [Octopus Deploy Samples Instance](https://samples.octopus.app) has grown expotentially since its creation in late 2019.  It has over 30 spaces, covering different deployment patterns or targets.  It takes time to spin up a new space and configure all the accounts, library variable sets, worker pools, etc.  Maintaining existing spaces takes more time.  Doing something as simple as rotating the cloud credentials on all spaces takes hours.  This post will walk through how the Customer Solutions team leveraged the [Octopus Deploy Terraform Provider](https://registry.terraform.io/providers/OctopusDeployLabs/octopusdeploy/latest/) to establish standards which solved those problems.

## Space Creation Woes

At Octopus Deploy, each team gets a unique sandbox for each of the major cloud providers.  One of the major benefits is security, I can have close to admin rights in the Customer Solutions, while read-only or limited rights in other team's sandboxes.  In our sandbox, we have tremdous flexibility, but that is both a blessing and a curse.  As it is very difficult to create and enforce standards when everyone has that level of permission.  

For example, we didn't have a good way share items such as resource group names in Azure, AMIs in AWSs, or regions in GCP between spaces.  Every time a space was created, the person would find a previous space they worked on and copy in those values.  If they were creating a sample using a cloud provider they hadn't used, they find a random space and use those values.  All the samples I created in AWS used the same VPC, security groups, and regions.  While another team member would have different VPCs, security groups, and regions.  On top of that, the resource names in Octopus wouldn't be the same across all spaces.  That, in turn, made maintenance much harder.

While that was a pain to deal with, it didn't cause _enough friction_ to change it.  It's not like we are creating spaces all the time.  And we do spot check items when we can.  That was until we were asked by our security team to move our AWS and Azure sandboxes.

That is when I started taking a hard look at the Octopus Terraform Provider.  All those above problems could be solved _and_ we'd learn a lot more about how the Terraform Provider worked.

## Space Standards

Each space on the samples instance is unique as each one shows how to use a different feature in Octopus Deploy.  What that meant was each space would get the same set of base resources to build on top of.  Those resources are:

- Accounts
    - GCP
    - AWS
    - Azure
- Worker Pools
    - GCP
    - AWS
    - Azure
- External Feeds
    - Docker
    - GitHub
    - Feedz (NuGet feed)
- Library Variable Sets
    - Notifications (standard messages for use with email and slack)
    - GCP
    - AWS
    - Azure
    - API Keys (to use when calling API scripts from Octopus)

You'll notice only worker pools are included, not specific workers.  Workers are created and destroyed on a schedule for each cloud provider.  We didn't want those managed by Terraform.

## Terraform Basics

At the start of this, my experience with Terraform was for a Proof of Concept.  Never to manage long-living production-level resources.  This section covers some of what I learned about Terraform and how it pertains to Octopus Deploy.

### Resource Ownership

Adding a resource such as a Infrastructure Account, Environment, or Library Variable to Terraform means Terraform owns the lifecycle of that item.  Any modifications made to that resource in the Octopus UI will be overwritten the next time the `terraform apply` runs for that space.

In addition, Terraform has total ownership over that resource.  One of the standards I wanted to enforce was every lifecycle's retention policy was set to 5 days.  But I wanted to let everyone add, edit, and delete lifecycles via the Octopus UI.  Terraform does not support that kind of dynamic resource management.  If I wanted Terraform to manage the lifecycles, each lifecycle would need to be added to the Terraform files.  In this use case, that was not acceptable.  I opted for an [API script](https://github.com/OctopusSamples/IaC/blob/master/octopus-samples-instances/api-scripts/enforce-lifecycle-policies.ps1) instead.

### State

Terraform uses a state file to determine which resources to create, update, and delete.  When a resource is appears in the Terraform file but not the state file it will attempt to create it.  By default the state file is stored in a subdirectory in the working directory.  The kicker is if you use Octopus Deploy to run those Terraform commands that working directory is deleted after each step runs.  What that means is that state file is automatically deleted.  The first time Octopus Deploy runs a `terraform apply` command everything will work because the resources won't exist.  But it will fail on subsequent runs because the resource already exists in Octopus Deploy.  That means you need to configure a [remote backend](https://www.terraform.io/language/settings/backends) to store your state file.  I opted for a secure S3 bucket.

### Importing pre-existing items into State

Almost all the items in the space standard list were new to each space.  Except for external feeds.  A number of spaces had a Docker external feed, a Github external feed or both.  I didn't want to create new external feeds and force everyone to update their deployment processes.  That'd be a waste of time.  I opted to import those pre-existing external feeds into the state before running the `terraform apply` step.

That wasn't as simple as running `terraform import`.  I wish it was.  I wanted my process to be able to run multiple times.  My script needed to:

1. Determine if the feed in question already existed in Octopus.  If it did not, then exit.
2. Determine if the feed in question already existed in the Terraform State file.  If it did, then exit.
3. Import the feed in question into State.

That is easier said than done.  The script would need to:

- Make an API call to Octopus to find the feed.
- Initialize the Terraform Workspace and login to AWS.
- Have Terraform pull the state file into memory so it could be searched.
- Loop through the state file in memory to see if the feed existed.
- Run the import command.

