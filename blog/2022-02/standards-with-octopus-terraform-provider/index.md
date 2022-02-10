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

The [Octopus Deploy Samples Instance](https://samples.octopus.app) has grown expotentially since its creation in late 2019.  It has over 30 spaces, which makes simple things like rotating an external feed password time consuming.  Creating a new space is also time consuming, as we have to copy over the same accounts, worker pools, external feeds, etc.  This post will walk through how the Customer Solutions team leveraged the [Octopus Deploy Terraform Provider](https://registry.terraform.io/providers/OctopusDeployLabs/octopusdeploy/latest/) to establish standards that solved those problems.

## Samples Instance Maintenance Woes

At Octopus Deploy, each team gets a unique sandbox for each of the major cloud providers.  Within our sandbox, everyone on the team has free reign which is both a benefit and a curse.  Each person ended up using whatever region was closest to them (for me it is US Central for Azure and GCP, US East 2 for AWS).  If that region didn't a virtual network, VPC, database server, etc., they'd be created.  Anytime a new space was created, the person creating the space would copy over values from an earlier example they worked on.  That was in addition to creating the cloud accounts, worker pools, external feeds, and other variable sets.

Maintaining the samples instance could be time-consuming.  For example, some of our samples used AMIs.  AWS is fond of deprecating AMIs, so anytime one was deprecated we had to find every sample that used it and update it.

While that was a pain to deal with, it didn't cause _enough friction_ to change it.  It's not like we are creating spaces all the time.  And we do spot check items when we can.  That was until we were asked by our security team to move our AWS and Azure sandboxes.  That is when I started taking a hard look at the Octopus Terraform Provider.

## Space Standards

Each space on the samples instance is unique as each one shows how to use a different feature in Octopus Deploy.  Projects between spaces were rarely alike.  The desire was to have the same set of base resources across all spaces.  Those resources are:

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

You'll notice library varible sets for each cloud provider in that list.  That was the second part of the space standards project.  We decided to create a set of base resources, VPCs or Virtual Networks, Subnets, Security Groups, etc. in a select number of regions in both the US and UK.  The names of those base resources are stored in those library variable sets so any sample on any space can leverage it.

## Using Octopus Deploy Runbooks to run Terraform Commands

I knew from the start I would be leveraging Octopus Deploy Runbooks to run the `terraform apply` or `terraform destroy` commands.  The question was, where would those runbooks exist?  I didn't want the runbooks to exist on the samples instance as that would turn into a strange "snaking eating the tail" scenario.  I opted to create a new instance, `samples-admin` to house these runbooks.  

The next step was coming up with a runbook structure to let me apply the Terraform files to all spaces on the samples instance.  The space list is dynamic, and I didn't want to have to add a new step or perform any additional maintenance to this process when a new space was added.  To solve this, I created the following runbooks:

**Enforce System Wide Standards** - runs scripts to enfore permissions on the everyone team, Octopus employees team, and Octopus managers team, then invokes **Enforce Space Standards** for each space on the samples instance.

![enforce system wide standards process](enforce-system-wide-standards-process.png)

**Enforce Space Standards** - accepts a Space-Id as a prompted variable, runs some API scripts to enforce retention policies, and runs the **Apply a Terraform Template** built-in step.  

![enforce space standards process](enforce-space-standards-proess.png)

The Space-Id prompted variable has `Spaces-1` as the default in the event I want to test a change quickly on a single space.  

![space id prompted variable](space-id-prompted-variable.png)

**Create New Space** - accepts a Space Name as a prompted variable, runs an API script to create a space, then calls the **Enforce System Wide Standards** runbook (using a prompted variable for a single space), to make sure permissions and resources are set up correctly for the new space.

![create new space process](create-new-space-process.png)

The scheduled trigger only invokes the **Enforce System Wide Standards** runbook.  Each time that trigger runs the following tasks are created:

- **Enforce System Wide Standards**
    - **Enforce Space Standards** (Spaces-1)
    - **Enforce Space Standards** (Spaces-2)
    - **Enforce Space Standards** (Spaces-3)
    - ...
    - **Enforce Space Standards** (Spaces-xxx)

:::hint
All scripts and terraform files used by these runbooks can be found in the [IaC GitHub Repository](https://github.com/OctopusSamples/IaC/tree/master/octopus-samples-instances).
:::

## Terraform Basics

At the start of this, my experience with Terraform was limited to Proof of Concepts.  This section covers some of what I learned about Terraform and how it pertains to Octopus Deploy.

### Resource Ownership

Adding a resource such as a Infrastructure Account, Environment, or Library Variable to Terraform means Terraform owns the lifecycle of that item.  Any modifications made to that resource in the Octopus UI will be overwritten the next time the `terraform apply` runs for that space.

One of the standards I wanted to enforce was every lifecycle's retention policy was set to 5 days.  Some of our spaces have a variety of lifecycles to demonstrate a particular feature.  There was no standard set of lifecycles.  Managing that standard was not possible via Terraform because:

- Resources have to be declared in the Terraform file.  We didn't want to lose the ability to add lifecycles via the UI.  Dynamically adding resources to Terraform files was a non-trival amount of work.  I didn't want some lifeycles managed by Terraform while others were not, that'd be very confusing.
- Terraform has total control over a resource.  I couldn't tell Terraform to only manage the retention policies.  

I opted for an [API script](https://github.com/OctopusSamples/IaC/blob/master/octopus-samples-instances/api-scripts/enforce-lifecycle-policies.ps1) instead.  

:::hint
You cannot have resources managed by Terraform and the Octopus UI.  It is either one or the other.  Make it clear which resources are managed by Terraform.  We opted to include the suffix `TF` on every name.
:::

### State

Terraform uses a state file to determine which resources to create, update, and delete.  When a resource is appears in the Terraform file but not the state file Terraform will attempt to create it.  

By default the state file is stored in a subdirectory in the working directory.  The kicker is if you use Octopus Deploy to run those Terraform commands that working directory is deleted after each step runs.  Which means that state file is automatically deleted.  The first time Octopus Deploy runs a `terraform apply` command everything will work because the resources won't exist.  But it will fail on subsequent runs because the resource already exists in Octopus Deploy.  

Storing the state file in a local directory isn't recommended anyway.  Any sensitive values are stored unencrypted in the state file.  A [remote backend](https://www.terraform.io/language/settings/backends) to store the state file is recommended.  I opted for a secured S3 bucket with encryption turned on.

In my initial version of the Terraform files, the state options were declared inline.  However, you can't use variables for state options.  In my ignorance, I decided to use Octostache, and have Octopus Deploy replace those values when the runbook ran.

```
terraform {
  required_providers {
    octopusdeploy = {
      source = "OctopusDeployLabs/octopusdeploy"
      version = ">= 0.7.64" # example: 0.7.62
    }
  }

  backend "s3" {
    bucket = "#{Project.AWS.Backend.Bucket}"
    key = "#{Project.AWS.Backend.Key}"
    region = "#{Project.AWS.Backend.Region}"
  }
}

provider "octopusdeploy" {
  # configuration options
  address    = var.octopus_address
  api_key    = var.octopus_api_key
  space_id   = var.octopus_space_id
}  
```

I did not like that.  If anyone but myself wanted to make an update, they had to know look at both the variables.tf and the main.tf files to make sure they had created all the variables.  It is too easy to forget.

In looking through [Terraform's documentation](https://www.terraform.io/language/settings/backends/configuration#command-line-key-value-pairs), I found it was possible to send in `-backend-config` arguments to the `terraform init` command.  Octopus Deploy has the ability to add additional arguments to the `terraform init` command.  Because I am using S3, the additional command line arguments are:

```
-backend-config="key=#{Project.AWS.Backend.Key}" -backend-config="region=#{Terraform.Init.S3.Region}" -backend-config="bucket=#{Terraform.Init.S3.Bucket}"
```

![terraform backend init in Octopus](terraform-backend-init-in-octopus.png)

Now my main.tf file looks like this:

```
terraform {
  required_providers {
    octopusdeploy = {
      source = "OctopusDeployLabs/octopusdeploy"
      version = ">= 0.7.64" # example: 0.7.62
    }
  }

  backend "s3" { }
}

provider "octopusdeploy" {
  # configuration options
  address    = var.octopus_address
  api_key    = var.octopus_api_key
  space_id   = var.octopus_space_id
}  
```

### Importing pre-existing items into State

Almost all the items in the space standard list were new to each space.  Except for external feeds.  A number of spaces had a Docker external feed, a Github external feed or both.  I didn't want to create new external feeds and force everyone to update their deployment processes.  I opted to import those pre-existing external feeds into the state before running the `terraform apply` step.

That wasn't as simple as running `terraform import`.  I wish it was.  I wanted my process to be able to run multiple times.  My script needed to:

1. Determine if the feed in question already existed in Octopus by invoking the Octopus API.  If it did not, then exit.
2. Determine if the feed in question already existed in the Terraform State file.  If it did, then exit.
3. Import the feed in question into State.

Invoking the Octopus Deploy API was not a problem for me.  I have [some experience working with the API](https://github.com/OctopusDeployLabs/SpaceCloner).  Everything else was new to me.  

Q: How do I provide credentials to `terraform init` for the S3 backend to work?  I want to do it the same way as the built-in steps in Octopus Deploy.
A: Octopus uses [environment variables](https://www.terraform.io/language/settings/backends/s3#credentials-and-shared-configuration) for credential backend.  In my case, I need to supply a value for `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`.

```PowerShell
$backendAccountAccessKey = $OctopusParameters["Project.AWS.Backend.Account.AccessKey"]
$backendAccountSecretKey = $OctopusParameters["Project.AWS.Backend.Account.SecretKey"]

$env:AWS_ACCESS_KEY_ID = $backendAccountAccessKey
$env:AWS_SECRET_ACCESS_KEY = $backendAccountSecretKey

terraform init -no-color
```

Q: How do I pull the terrform state into memory so it could be searched?
A: This is accomplished by running the `terraform show` command with the argument `-json` specified.  Then it is a matter of converting that json into an object PowerShell can search through.

```PowerShell
$currentStateAsJson = terraform show -json
$currentState = $currentStateAsJson | ConvertFrom-Json -depth 10
```

Q: What is the structure of the state file?  How do I search through it?
A: This took a bit of trial and error but I ended up with:

```PowerShell
function Get-ItemExistsInState
{
	param 
    (
    	$currentState,
        $itemTypeToFind,
        $itemNameToFind
    )
    
    foreach ($item in $currentState.Values.root_module.resources)
    {
        Write-Host "Comparing $($item.Name) with $itemNameToFind and $($item.type) with $itemTypeToFind"
        if ($item.name.ToLower().Trim() -eq $itemNameToFind.ToLower().Trim() -and $item.type.ToLower().Trim() -eq $itemTypeToFind)
        {
            Write-Host "The item already exists in the state"
			return $true
        }
    }   
    
    return $false
}
```

Q: What is the syntax to import items into Terraform state?
A: The command to import an item into state is `terraform import [address] [id]`.  The address is what is defined in the terraform file.

For the resource defined as:

```Terraform
resource "octopusdeploy_feed" "github" {
  name = "GitHub Feed TF"  
  feed_type = "GitHub"
  feed_uri = "https://api.github.com"
  is_enhanced_mode = false
}
```

The address is **octopusdeploy_feed.github**.  The id was a lot easier to determine, as that is the ID of the item in Octopus Deploy.  For example `Feeds-1070`.  

:::hint
You can see a sample of the script I wrote in our [Samples IaC GitHub Repository](https://github.com/OctopusSamples/IaC/blob/master/octopus-samples-instances/api-scripts/import-into-state-example.ps1)
:::

### Passing in variable values

With Terraform you define and use a variable like so:

```Terraform
terraform {
  required_providers {
    octopusdeploy = {
      source = "OctopusDeployLabs/octopusdeploy"
      version = ">= 0.7.64" # example: 0.7.62
    }
  }

  backend "s3" { }
}

variable "octopus_address" {
    type = string
}

variable "octopus_api_key" {
    type = string
}

variable "octopus_space_id" {
    type = string
}

provider "octopusdeploy" {
  # configuration options
  address    = var.octopus_address
  api_key    = var.octopus_api_key
  space_id   = var.octopus_space_id
}
```

There are three options for passing in values to those variables.

- Command line argument: `-var="octopus_address=https://samples.octopus.app"` or `-var="octopus_address=#{Octopus.Web.ServerUri}"`
- Variable file: naming a file with the extension `.tfvars` and populating it with values.  For example `octopus_address = "https://samples.octopus.app"` or `octopus_address = "#{Octopus.Web.ServerUri}"`
- Environment variables: `export TF_VAR_octopus_address=https://samples.octopus.app` or `$env:TF_VAR_octopus_address = https://samples.octopus.app`