---
title: Shaping Config as Code for variables
description: Learn how customer feedback informed updates to Configuration as Code.
author: dylan.lerch@octopus.com
visibility: public
published: 2022-09-28-1400
metaImage: blog-post-cac-variables.png
bannerImage: blog-post-cac-variables.png
bannerImageAlt: Octopus UI and a config file in a text editor
isFeatured: false
tags:
- Product
- Engineering 
- Configuration as Code
---

We launched [Configuration as Code (Config as Code)](https://octopus.com/blog/octopus-release-2022-q1) in March 2022, and have continued adding new features based on customer feedback. One of the most requested features was variables for Config as Code.

As of [Octopus 2022.3](https://octopus.com/blog/octopus-release-2022-q3), you can store your non-sensitive variables in Git alongside your deployment process and deployment settings.

In this post, I cover the changes we made and dive into the design of the variables Octopus Configuration Language (OCL) schema.

## Why variables is the first major upgrade to Config as Code

Since the release of Config as Code in March 2022, we've been listening to everyone's feedback to continue improving our support.

While there have been many feature requests, there was a clear theme: _Config as Code feels incomplete without variables in the Git repository_. 

Variables was first, but we'll continue listening to your feedback and evolving Config as Code to meet your needs. 

## Config as Code becomes more powerful

The first release of Config as Code had good support for version-controlled deployment processes and settings, but process changes that needed complementary variable changes required careful coordination. If you changed the wrong value, or deleted the wrong variable, it could disrupt other deployments. 

With project variables now in the Git repository, the Config as Code feature and branching are even more powerful. You can make major changes to your deployment process and variables together on a feature branch without impacting anyone else until you're ready to merge your changes.

Octopus now has branch context on the project variables page. When setting action scopes, it pulls the list of actions from the selected branch (rather than only showing actions from the default branch).

![Screenshot of Octopus project variables page showing variable action scopes expanded showing 'Script for new feature' selected](git-variables-action-scoping.png "width=500")

This makes action scoping much more useful in Git projects. You can add a new step on your feature branch, scope a variable to this step, and merge these back to your default branch as a single block of configuration when you're ready.

### Support for sensitive values

For now, *sensitive variables will remain in the Octopus database*. On the project variables page, we make this clear by showing a database icon next to all sensitive values.

![Screenshot of Octopus project variables page showing database icon to the left of masked sensitive variable placeholder](git-variables-secrets.png "width=500")

Regardless of the selected branch, commit, or tag you'll always be viewing and modifying the single shared set of sensitive variables. We're investigating the best way to securely store sensitive variables in Git. We hope to add support for this at some point in the future, but for now sensitive values remain in the database.

## Getting started

After you upgrade to a [supported version](https://octopus.com/downloads), the project variables will automatically be migrated alongside your deployment process and deployment settings when converting the project to Git. You don't need to complete any additional steps.

For existing Git projects, you need to migrate your variables to Git manually.

### Migrating variables to Git

We initially planned to automatically migrate the variables on the next commit for everyone who upgraded. After some testing of this approach, however, it was clear this wouldn't work for many teams. The potential for disruption was too great, especially for cloud customers who receive frequent updates to their instances.

Instead, we opted to have a transition phase, so everyone can perform the migration at a time that suits them. If you have an existing Git project with variables that haven't been migrated, you'll see a banner on the project page.

![Screenshot of Octopus project variables page showing database icon to the left of masked sensitive variable placeholder](git-variables-migration.png "width=500")

Clicking the **MIGRATE VARIABLES TO GIT** button shows a wizard that guides you through the migration. 

:::hint
For more detailed documentation on the migration process, check out the [migrating variables to Git documentation](https://oc.to/ConfigAsCodeVariables).
:::

#### Transitioning to automatic migration

This transition phase has resulted in 3 possible project states:

- Git projects with Git variables
- Git projects with database variables
- Database projects

This adds additional complexity to the codebase, and requires time and effort that could be used for new features. With this in mind, *we're not going to support this transition state forever*. We'll eventually remove support for Git projects with variables in the database, so we recommend migrating your variables as soon as practical.

### Route changes

With the project variable set now split between your Git repository and the Octopus database, Git projects have 2 project variable routes:

- `/api/{spaceId}/projects/{projectId}/{gitRef}/variables`: Use this route for all variable types that aren't sensitive. There's a separate set of variables for each [Git reference](https://git-scm.com/book/en/v2/Git-Internals-Git-References), and specify this reference in the `{gitRef}`. While you can read any type of reference, commits and tags are immutable, so you can only write to branches.
- `/api/{spaceId}/projects/{projectId}/variables`: Use this route for your sensitive variables. These remain in the database and are never written to your Git repository.

The original variable route (`/api/{spaceId}/variables/variableset-{projectId}`) will continue to work for projects with variables in Git, but it will only return the sensitive values. We recommend always using the new project scoped routes.

These don't apply if you only use Octopus through the UI. The project variables page will write the values to the correct locations for you.

## The variable OCL schema

An ongoing goal for Config as Code is to keep the Octopus UI fully functional for Git projects. We want to provide a great experience for those who edit OCL directly in a text file.

In the first version of Config as Code, we took our existing deployment process and deployment settings resource models and translated them directly to OCL to create the schema. This worked well for these resources, but we had to take a different approach with variables.

Variables are passed through the API and persisted as a single-level array of variables. Multi-value variables are persisted as completely separate variables with the same name. Had we just written this directly into the Git repository, the OCL would have looked something like this.

```hcl
variable "DatabaseName" {
    value = "AU-BNE-TST-001"
    scope = {
        environment = ["test"]
    }
}

variable "DatabaseName" {
    value = "AU-BNE-001"
    scope = {
        environment = ["production"]
    }
}

variable "DeploymentPool" {
    type = "WorkerPool"
    value = "production-pool"
    scope = {
        environment = ["production"]
    }
}

variable "DeploymentPool" {
    type = "WorkerPool"
    value = "non-production-pool"
}

variable "DatabaseName" {
    value = "AU-BNE-DEV-001"
    scope = { 
        environment = ["production"]
    }
}
```

This was functional and the UI worked as expected, but the OCL editing experience wasn't ideal. There were repeated variable names and types, values for the same variable were easily separated, and it created unnecessary nesting. Instead, when serializing to OCL, we merge values for the same variable together, flatten out the scopes, and everything is a lot cleaner.

```hcl
variable "DatabaseName" {
    value "AU-BNE-TST-001" {
        environment = ["test"]
    }

    value "AU-BNE-DEV-001" {
        environment = ["development"]
    }

    value "AU-BNE-001" {
        environment = ["production"]
    }
}

variable "DeploymentPool" {
    type = "WorkerPool"

    value "non-production-pool" {}

    value "production-pool" {
        environment = ["production"]
    }
}
```

We took a completely different approach to OCL serialization to achieve this, but we're happy with the result. It makes OCL easier to reason with and provides a great editing experience. This gives us options for how we define OCL schemas in the future, and will feed into enhancements we plan to make to the persistence and API layers.

<!-- :::hint
For a more detailed information and examples, check out the [variable OCL schema documentation](https://to-do).
::: -->

## What's next?

The latest version of Config as Code with Git variables is rolling out to Cloud and available on on-premises when you [download the 2022.3 release](https://octopus.com/downloads). After your instance is up-to-date, you can migrate your variables to Git as soon as you're ready.

We'd love to hear your feedback and learn what features you want next via our [Config as Code feedback form](https://oc.to/CaCEAPFeedbackForm).

<span><a class="btn btn-success" href="https://oc.to/CaCEAPFeedbackForm">Provide feedback</a></span>

Happy deployments!
