---
title: Feature Branch Environments with Kubernetes and Octopus Deploy
description: Creating dynamic environments inside of Kubernetes for feature branches with Octopus Deploy
author: bob.walker@octopus.com
visibility: public
published: 2024-12-31-1400
metaImage: 
bannerImage: 
bannerImageAlt: 
isFeatured: false
tags: 
  - Product
  - Kubernetes  
  - Git
---

One of the challenges with short-lived feature branches is testing and getting feedback before merging into the main branch. Creating and deploying to short-lived infrastructure solves this problem. But, it can be challenging to build and eventually destroy that infrastructure. In this blog post, I will discuss how to accomplish that with Kubernetes and Octopus Deploy.

## Defining the problem

A core principle of continuous delivery is that the main branch must always be ready to deploy to production. It should match production or represent an upcoming desired production state (about to be deployed).  

To accomplish that, you’ll need to incorporate a branching strategy. Some common branching strategies include [Feature Branch Workflow](https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow), [Trunk-based Development](https://trunkbaseddevelopment.com/), or [GitHub Flow](https://docs.github.com/en/get-started/using-github/github-flow). All of those strategies leverage short-lived feature branches.  

While working on a code change in a feature branch, many developers ask, "Will this work as the business expects?"  The only way to know is to deploy it to a location where it can be verified.

One of the default behaviors of Octopus Deploy that I’m not a fan of is that it puts all the environments (Dev, Test, Staging, and Production) in a single channel. It helps remove a barrier to the first deployment from an onboarding perspective (which is a good thing). However, that leads to a deployment pipeline that looks similar to this:

![](first_deployment_pipeline.png)

The fundamental problem with that pipeline is that you must merge to the main branch before deploying to development to get feedback. This poses the following problems when you want to get feedback:

- Half-finished changes are merged into the main branch to get that feedback.
- Very rarely will everything work as expected on the first merge, resulting in dozens of deployments.
- This means the main branch is likely undeployable, violating the core principle that it must always be ready to deploy to production.

A good first step to solving this problem is to create two channels or pipelines. I already had this process in place before working on this article.

- Default: Deploy to Development Only
- Release: Deploy to Test -> Staging -> Production

Feature branches are deployed to the default channel. The main branch is deployed to the release channel.

![](second_deployment_pipeline.png)

That works when you have one feature branch at a time. However, many applications will have multiple active feature or hotfix branches. What is needed is the ability to create temporary environments.

![](ideal_deployment_pipeline.png)

We will create temporary environments for this blog post in Kubernetes and SQL Server.

The rules for this process are:

1. Feature branches
   1. Deploy to temporary environments.
   2. The namespaces and database are temporary, and the K8s cluster and SQL Server are static.
   3. Every code check-in on a feature branch triggers a new build. 
   4. The built artifact is only deployed to temporary environments. Tag each build artifact with a pre-release tag.
   5. Delete the temporary environment when a pull request is closed.
2. Main Branch
   1. The main branch deploys to static environments, at least one test environment, and production.
   2. A new build occurs for every code check-in on the main branch.  
   3. The built artifact is deployed to static testing environments and production.

## Build from main once, deploy everywhere

At first glance, this process violates the "build once, deploy anywhere" principle because I’m rebuilding after merging into the main branch. However, that build from the main branch is required. 

Consider this scenario:
- A feature branch is created from the main branch.
- Shortly after, a hotfix branch is created from the main branch.
- The hotfix branch is merged into the main branch.
- The feature branch and hotfix branch didn’t touch the same code. 
- The feature branch is merged into the main branch shortly after the hotfix branch.

![](branching_and_main.png)

We want the main branch to represent production. After all changes have been merged, they must be tested together.  

Rebuilding from the main branch after each check-in follows "build once, deploy anywhere." The build artifact from the main branch is reused for each testing environment and production. The critical difference is that the built artifacts from the feature branches are thrown away after testing. Those are never supposed to go to production.

## Configuration

I will use Kubernetes hosted on Azure (AKS) to host my application and Azure SQL for my database. We will re-use the same AKS Cluster and Azure SQL Server for the dynamic infrastructure. I’m using GitHub Actions as my build server.

**Important:** This workflow should work with any application host you can re-use, such as ECS clusters, Windows, or Linux. It is much more difficult when you cannot re-use the same app hosts, such as Azure Web Apps or other PaaS-hosted applications.  

### Versioning Strategy

The versioning strategy of the build artifacts is a critical aspect of this workflow.  

- Feature Branches: Have a pre-release tag, for example, [4.0.70-demo.2](https://hub.docker.com/layers/bobjwalker99/trident/4.0.70-demo.2/images/sha256-d9ac568c45cfea37a55039371c141f293b703e0cf59e98adf0e6add512b633eb?context=explore). The .2 at the end represents the build number.
- Main Branch: No pre-release tag, for example, [4.0.69](https://hub.docker.com/layers/bobjwalker99/trident/4.0.69).

The build server is responsible for creating the version number for the build artifacts. I will cover that later in the build server configuration.


### Octopus Deploy Configuration

The following items will need to be updated in Octopus Deploy:

- Lifecycles
- Project
  - Config-as-Code
  - Channels
  - Runbooks
  - Variables
  - Deployment Process

#### Lifecycles

Configure two lifecycles, one with development only, and one with the remaining environments.

**Information:** I configure both lifecycles to auto deploy releases for the first environment. This is so I don’t have to add a "deploy release" step in my build servers. As an added bonus, I don’t have to click deploy after creating a release via the UI either.

![](lifecycles.png)

My dynamic infrastructure will exist in the development environment, while the environments in the release lifecycle will be static.


#### Config-as-Code

For my configuration, I am leveraging the config-as-code feature with Octopus Deploy. The Octopus Configuration and GitHub Actions are stored in the same [git repository](https://github.com/BobJWalker/Trident) as the application source code.

![](github_repo_overview.png)

I’m doing this because when I create the release for the feature branch, I have access to the branch name and other important information.

![](feature-branch-name-with-releases.png)

In addition, if I need to make any changes to _how I deploy_ with the feature branch, I can do all the work in the branch and include the code changes, database changes, and deployment process changes in the same pull request.  


#### Channels

My project has two channels, one for each of the above lifecycles. The channels have version rules to enforce the pre-release tag rule from the versioning strategy section.

![](channels.png)

- Default Lifecycle allows any pre-release tag: `^\[^\\+].\*`
- Release Lifecycle doesn’t allow any pre-release tags: `^(|\\+.\*)$`

#### Runbooks

I created two runbooks, one to create the infrastructure and another to destroy the infrastructure. My application has two components: a web application and a database backend. The create infrastructure creates a namespace for the web application in Kubernetes and the databases in Azure SQL.

**Information:** You’ll notice the "Check Database" and "Run Script for the databases."  Flyway requires the check database to perform the deployment. The run-a-script step for the databases ensures I can run a basic "select 1" query on the database. The process of creating infrastructure will likely be different.

![](create_infrastructure_backend.png) 

The destroy infrastructure runbook does the opposite; it deletes the database and the namespace.

![](destroy_infrastructure.png)


#### Variables

The variables are the key to this entire process in Octopus Deploy. You have to come up with a naming convention that leverages the feature branch name for this to work. I’ll include a formatted feature branch name for anything specific to the development environment.

**Important:** You must format the branch name, as SQL Server and K8s might not like specific characters.  

I’ve highlighted the key variables in my process below.

![](variables.png)

The variables are:

- **Project.Release.Branch.Name**: This pulls the branch name from CaC for the deployment process for CaC. The value is: #{Octopus.Release.Git.BranchName}.  
  - Because my version of Octopus does not have CaC for runbooks, I have to use [prompted variable](https://octopus.com/docs/projects/variables/prompted-variables) for my two runbooks. The default value for the prompted variable is `main`.
- **Project.Formatted.Branch.Name**: This variable will remove all the invalid characters and format it for use in other variables. The value is `#{Project.Release.Branch.Name | Replace "/" "-" | ToLower | Replace "refs-heads" ""}`.
- **Project.K8s.Namespace**: Used by the deploy steps and create namespace step. For the development environment, I’ve included `#{Project.Formatted.Branch.Name}`.
- **Project.Database.Name**: Used by the database deploy steps and connection strings to determine the application's database name. For the development environment, I’ve included `#{Project.Formatted.Branch.Name}`.
- **Project.Database.Check.Name**: Used by the Flyway steps for deployments as a temporary database. If you aren’t using Flyway, you won’t need this variable. For the development environment, I’ve included `#{Project.Formatted.Branch.Name}`.
- **Project.URL.Local**: Specifies the domain for Kubernetes Ingress. I had to modify my Kustomize overlay file in Dev and inject [Octostache](https://octopus.com/docs/projects/variables/variable-substitutions). While I preferred not to do that, this was an appropriate compromise, as it was limited to the dev overlay only.


#### Deployment Process

The deployment process calls the create infrastructure runbook and waits for it to complete. I injected the branch name from CaC into the prompted variable.

![](run_a_runbook_step_process.png)

**Important:** I call this runbook for every environment because each night, I tear down all my infrastructure in Azure to save costs. In your deployment process, consider configuring this step to run for the development environment only.

Any steps that interact with the Kubernetes cluster use the namespace variable.

![](k8s_steps_deployment_process.png)

Any steps interacting with the database will use the connection string variable, which uses the database name variable.

![](database_steps_deployment_process.png)


### GitHub Actions

I’m using GitHub Actions as my build server. I have two GitHub Actions:

- Build - builds the artifacts, pushes them to Octopus / Docker Hub, and creates the release.
- PR Closed - Invokes the destroy runbook in Octopus Deploy when a PR is closed.


#### Build Action

For my build action, I wanted to build on meaningful code changes for specific branches and provide the capability to trigger the action manually. In addition, I prefer to leverage environment variables to eliminate as many magic strings as possible.  

![](build_gh_action_start.png)

This build calculates the version number. I’m using [gittool’s gitversion action](https://github.com/GitTools/GitVersion) to handle that work. That action creates several output variables you can leverage in later steps. I primarily use the env.GitVersion\_SemVer output variable.

The next step shows the first use of that output variable. I like to tag the version number on the main branch using the [update tag GitHub action](https://github.com/marketplace/actions/update-tag). That particular steps only run because of the if condition.

![](gh_actions_calculate_version.png)

That output variable appears when I create the tag for my docker image.

![](gh_action_create_docker_image.png)

The final step in my action is to create a release in Octopus Deploy.

![](gh_action_create_release.png)

GitHub’s syntax can be hard to decipher; here is what each of those options means.

- **Project Name**: The name of the Project in Octopus Deploy. Pull from the environment variables defined earlier in the action. `${{ env.OCTOPUS\_PROJECT\_NAME }}`
- **Channel**: The name of the Channel in that Project. When on the main branch use the release Channel, otherwise use the feature branch Channel. `${{ github.ref == 'refs/heads/main' && env.OCTOPUS\_RELEASE\_CHANNEL || env.OCTOPUS\_FEATURE\_BRANCH\_CHANNEL }}`
- **Release Number**: The release number in Octopus Deploy. Uses the same gitversion output variable as before. `"${{ env.GitVersion\_SemVer }}"`
- **Package Version**: The version of each of the packages for the release. Uses the same gitversion output variable as before. `"${{ env.GitVersion\_SemVer }}"`
- **Git\_ref**: This is required for Projects with config as code enabled. Specifies the branch. `${{ (github.ref\_type == 'tag' && github.event.repository.default\_branch ) || (github.head\_ref || github.ref) }}`
- **Git\_commit**: This is required for Projects with config as code enabled. The git commit that triggered this action.   `${{ github.event.after || github.event.pull\_request.head.sha }}`

#### Pull Request Action

The Pull Request action has a single step: invoke the destroy infrastructure Runbook in Octopus Deploy when the PR is Closed.

![](gh_action_pr_closed.png)

The variables parameter is the critical parameter, which tells the runbook which feature branch to destroy. I wanted to invoke this runbook manually for testing, which is why you see ${{ github.head\_ref || github.ref }}. That tells the GitHub action to use the branch that created the PR. It uses the specified branch when manually invoking that action if it is not in a PR.

### Kustomize Overlays

I had to make a minor modification to the development overlay. I wanted a custom domain for each feature branch of my ingress in Kubernetes. The easiest way to accomplish that was to inject the value using Octostache. As this overlay is only used for development, that was an appropriate compromise. Helm charts and raw manifest files have other options that don’t require Octostache in the file. Kustomize has a limitation in that we can’t inject structured variable replacement after the file has been transformed.

![](kustomize_overlays_dev.png)

All the other overlays do not have Octostache, as those are static environments.

![](kustomize_overlays_prod.png)


## See it in action

On my AKS Cluster, I have the following namespaces:

![](aks_before_namespaces.png)

On my Azure SQL Server, I have the following databases: 

![](azure_sql_before_databases.png)

When I create a new branch called feature/blog-post and check in a change a build will be triggered.  

![](gh_action_running.png)

Once the action is completed, I see version 4.0.71-blog-post.1 was created.

![](gh_action_complete.png)

Once that is done, I see two new databases.

![](azure_sql_after_databases.png)

As well as a new namespace.

![](aks_after_namespaces.png)

I create and merge my pull request.

![](gh_pull_request.png)

Once the PR is approved and closed, the GH Action triggers the runbook. 

![](gh_actions_destroy_infrastructure.png)

The namespace and databases are deleted and now we an move onto the next feature branch!

## Limitations

There are known limitations and shortcuts used when creating this process.  

- It reuses a pre-existing application host (AKS K8s Cluster) and database server (Azure SQL). Creating and destroying new infrastructure makes it harder, as you’ll need a mechanism to tell Octopus to deploy only to new infrastructure. Use [Matt Casperson's](https://octopus.com/blog/feature-branch-web-apps) excellent article as a reference.
- There are a few environmental configuration items. If you leverage a configuration or secret store, additional configuration is required.
- The application has no external dependencies. If your application has external dependencies, your options are:
  - Point to services running on a static testing environment.
  - Create all the external dependencies for each feature branch.
- The build server (in my case, GitHub Actions) is more tightly coupled with the deployment server. It is aware of different channels and runbooks.
- The deployment dashboard in the Octopus Project only shows the last three successful deployments to an environment. If I had many feature branches in flight, I’d need to go to the project's releases page to see the latest deployment for my branch.


## Conclusion

Using the above mentioned process, we can create and destroy dynamic testing infrastructure for each feature branch. Depending on your usage of Octopus Deploy, it might require only a few modifications to your existing configuration. Chances are you already have a lot of this configuration in place for different environments. That is what makes me so excited about it. Some minor modifications to an existing process and you can get it working. Before working on this article, every feature branch was deployed to a static development environment. That means I already had the channels and runbooks configured. The only thing I needed to modify was to add some new variables and tweak my Kustomize overlay file.

What is even more exciting is you can apply this pattern to other application types if you can reuse other infrastructure, such as ECS clusters, Windows servers, or Linux servers.

You can find the [git repository on GitHub](https://github.com/BobJWalker/Trident). In addition, you can view the process on [my personal cloud instance](https://bobjwalker.octopus.app/) (login as a guest).

Please do note the limitations with what is proposed in this article.  If you implement this process, use this article as a baseline. Take what works for you and modify where appropriate. You’ll likely need to iterate a few times before it works.

Until next time, Happy Deployments!
