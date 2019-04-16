---
title: Leveraging Multi-Tenancy to carve out team and developer sandboxes
description: The software we write often has dependencies on other team's applications.  Learn how you can use the multi-tenancy feature in Octopus Deploy to carve out sandboxes for each team.
author: bob.walker@octopus.com
visibility: public
published: 2019-04-18
metaImage: 
bannerImage: 
tags:
 - Multi-Tenancy
---

Recently I had a chance to meet with a customer to talk about their development and deployment process.  Full disclosure: I'm going to paraphrase the meeting a bit, so my apologies to them if I grossly simplify their concerns.  The customer had a common scenario.  They have three teams of developers and four "main" applications.  There are some dependencies between the four main applications.  Application A might call Application B which calls Application C and so on.  There isn't a clear "line in the sand" on which team can work on which application.  At any given point, two teams might be working on the same application.  For example:

- Team A is adding a new feature to Application A
- Team B is making some bug fixes to Application A
- Team C is adding a new feature to Application B

Their concern was how Team A could push up changes to Application A for their QA engineers to test and business owners to verify without stepping all over Team B's work.  As an added kicker, how can Team C make changes to their application without stopping Team B from pushing a set of bug fixes through?

I ran into this scenario multiple times at different jobs before working at Octopus Deploy.  The tooling, and the collective lack of knowledge, at the time, led to some, let's say, unusual solutions.  Team C was told that whatever changes they made had to be backwards compatible from the start.  Hide it behind a feature flag if you must.  You cannot stop work for Teams A and B.  Team A was told to hold off pushing their changes until Team B had finished their work.  That is just three teams.  I've worked for companies with 10+ development teams.  Imagine trying to juggle all that!

The final kicker is sometimes a developer will be working on a spike or another change, and they want to get feedback from someone.  This feedback could come from another developer, or it could come from a business owner.  In the past when I did this, I had the developer and/or business owner connect directly to my machine.  Connecting directly to my machine was a significant pain on my end as I could be in the middle of tweaking something else and the code might not be in a usable state.  It would've been nice to have someplace I could push the change without breaking everyone.

A much more ideal solution is each team, as well as developers, get their own "sandbox" hosting all of the applications.  

- Team A can push a new feature to their sandbox for testing without stepping all over Team B.  
- Team C can push changes to their sandbox for testing and not break Teams A and B.  
- Team A and B are not aware of Team C's changes because they are pointing to their own copy of Application B.  
- Once Team C's changes have been verified, then Teams A and B will need to update their sandbox to get the latest and greatest changes.

In this article, I am going to walk you through how to configure that using the multi-tenancy feature in Octopus Deploy.

**Please Note:** You have to have your entire deployment process automated for this to work.  You can't have your code (C#/JS/TS/PHP/etc.) being deployed through Octopus Deploy while your databases are deployed manually.  

## Configuring Octopus Deploy

Before diving into the configuration, let's take a step back and look at some use cases.  Hopefully, at the end of the article, we meet all of them.  

- As a developer, I want to be able to make and push changes to Application B without affecting the work of other teams working on Application A.
- As a developer working on Application A, I want to be able to pull changes made to Application B in once they are completed and verified.
- As a developer, I always want to be able to test what is currently in production in a non-production environment.
- As a developer, I want to be able to make changes and push them up for verification from a business owner without affecting my team.
- As a developer on Application A, I want to know what version of Application B I am connecting to and testing against.
- As a developer, I want to be able to push a hotfix directly to staging without having to go through development or testing first.
- As a developer, I want to be able to push my team's changes to a testing area where customers can provide feedback.  Getting customer feedback isn't going to happen all the time, but I would like that opportunity.

## Tenant Tags

I am going to create a tenant tag set called `Tenant Type` for grouping and organizational purposes.  Based on my use cases, I am going to need three tags in the `Tenant Type` tag set.

- Release (main or master and hotfix)
- Teams (for the team to test with)
- Person (each developer)

![](tenant-tags.png)

## Global Variables

For my instance, I have two variable sets.  `Global` is to be used across all the projects.  `OctoFx` is to be used for any OctoFx projects.  

![](variable-sets.png)

I am going to host all my tenants on the same SQL Server and Web Server used in each environment.  I have one web server for `Development` and two web servers in the `Testing` environments.  That single `Development` web server will host not only the main website but also all my tenant websites.  Each tenant will get their subdomain.  The naming convention I am going to use for tenants assigned to the Teams or Person `Tenant Type` will be [EnvironmentShortName][TenantShortName][ApplicationName].OctopusDemos.com.  For Bob's copy of the OctoFx application in Development, the name will be DevBobOctoFx.OctopusDemos.com.  For release `Tenant Type` the naming convention will be [EnvironmentShortName][ApplicationName].OctopusDemos.com.  The master version of OctoFx in Development will be DevOctoFx.OctopusDemos.com.

I am using the same SQL for both Dev and Testing, so my database names will be rather odd looking at first.  [EnvironmentPrefix]-[TenantShortName]-[ApplicationName] for tenants assigned to the Teams or Person `Tenant Type`.  Bob's copy of the OctoFx database in `Development` would be `d-Bob-OctoFx`.  Tenants assigned to the Release `Tenant Type` will be [EnvironmentPrefix]-[ApplicationName].  The master version of OctoFx in `Development` will be `d-octofx`.

First I am going to create a variable in Global to store the environment short name as well as the environment prefix variable.  

![](environment-short-name-global-variable.png)

**Please Note:** I like to use the naming scheme [VariableSet].[Component].[Name] to name my variables.  The naming scheme makes them easier to find when trying to reference them.  Moreover, when looking at all variables for a project, they group a lot better.

Then I am going to jump over to the `Variable Templates` tab and add a new template.  I will need to populate this when I create my tenants and assign them to the projects.

![](variable-sets-templates.png)

Now I can jump back over variables tab and add two variables, one for the subdomain name and another for the database name.  You can scope variables to tenant tags.

![](variable-set-with-scoped-variables.png)

You will notice I am referencing a variable in there called `Project.Application.Name`.  That variable will be set at the project level.  Variable replacement is done at runtime.  Variable replacement at run time is needed because quite often we don't know the value until the step is being run on the machine (machine name, output variables, etc.).  

The `OctoFx` variable set is nothing super complex.  It stores some database values so it can build up a connection string.  

![](octofx-variable-set.png)

## Infrastructure

As I stated in an earlier section, I am going to be deploying to a SQL Server and multiple web servers.  I will be using a worker to deploy to SQL Server.  Only the web servers will have tentacles on them.  I need to update the tentacle registration to allow for multi-tenant deployments.  First, we need to tell Octopus Deploy to allow for both tenant and multi-tenant deployments to each web server.

![](machine-settings-tenant.png)

When we do that a new section appears on the screen allowing us to select which tenants to allow.  My thinking is:

- Production: Release `Tenant Type`
- Staging: Teams and Release `Tenant Type`
- Testing: All `Tenant Type`
- Development: All `Tenant Type`

Production only accepts releases or hotfixes.  Staging will be limited like Production, as well as teams because of the above use case "As a developer, I want to be able to push my team's changes to a testing area where customers can provide feedback."  Quite often I see setups where `Development` and `Testing` are only available to internal users.  However, `Staging` needs to match production.  Typically that includes external access.  

I will add the appropriate `Tenant Type` tenant tags to the machine registration.

![](infrastructure-with-tenant-tags.png)

When I am finished updating my machines the overview screen will look like this:

![](infrastructure-overview-with-tenant-tags.png)

## Projects

Before configuring the tenants, I want to take a look at the `OctoFx` application I will be using as my example.  OctoFx consists of two component, a database, and a web interface.  Each component has its deployment project in Octopus Deploy.  Not every change made to OctoFx requires the full application to be deployed.  

![](octofx-overview.png)

Adding an index to the database shouldn't require the website to be deployed.  Updating the CSS shouldn't require the database to be deployed.  Separating them helps to save time.  That really comes in handy when you have to deploy a hotfix.  Having to wait on a database deployment to finish when you really need to fix the website is maddening.  

Each project only has to focus on one thing.  The database project only worries about database deployments.  Make note; this project was created with the assumption that the database did not exist in that particular environment.  It will create a new database and run the necessary scripts to add the schema, data, and users.  

![](octofx-db-project.png)

Whereas the web deployment only has to worry about steps for web deployments.  Just like the database project, it was written with the assumption the website didn't already exist.  The deploy to IIS will create the website on the server.  It then has a step to add the website to DNS.  

![](octofx-web-overview.png)

There are cases when you want to deploy the entire stack.  That is where the "Traffic Cop" project comes in.  It uses the deploy a release step to deploy the components in the correct order.

![](octofx-trafficcop-overview.png)

The build server is responsible for creating the releases for each component.  A person will come in and create the release for the traffic cop project.  Make note that the lifecycle allows you to deploy straight to staging, skipping `Development` and `Testing`.  Skipping those environments is to handle those cases when individual components are pushed to `Testing` manually.

**Please note:** The projects on my example instance were created with the assumption the database and/or website didn't exist.  Each project performs the necessary steps to get the component up and running.  For this sandbox approach to work your projects will need to be created with a similar concept.  You don't have to repeat the exact process I used in my projects.  

The only other significant adjustment you will need to make is configuring your projects to allow for multi-tenancy deployments.  That configuration can be found by going to the settings section on the project and changing the multi-tenant setting.  In my example, I am choosing to require a tenant for deployments.  If you were transitioning a project over to multi-tenancy the first time, it would make more sense to allow for deployments with and without a tenant.  

![](project-multi-tenant-settings.png)

## Branching, Lifecycles, and Channels

Now is as good as time as any to discuss branching strategies.  The branching strategy will correspond to a lifecycle and project channel in Octopus Deploy.  I am going to have my build server append a pre-release tag to any package build from non-release branch (this could be the master branch or a hotfix branch).  The build server will also determine the tenant and channel based on the branch name.  

- Release branch (release/*): Deploy to Development -> Testing -> Staging -> Production.  95% of all production releases will go through this lifecycle.
- Hotfix branch (hotfix/*): Can skip Development and Testing and go straight to Staging and then Production.  5% of all production releases should go through this lifecycle.  This allows for a clear path to production while providing the ability to test.
- Team Branch ([TeamName]/*): Can only go to Development -> Testing -> Staging.  This branch will have a pre-release tag appended to the package name.
- Developer Branch ([DeveloperName]/*): Can only go to Development -> Testing.  This branch will have a pre-release tag appended to the package name.

In Octopus Deploy I created the following lifecycles.

![](lifecycles-overview.png)

In each project, I added the following channels.

![](channeloverview.png)

Each channel is using has a version rule on the for the pre-release tag for the package.  For Team and Developer deployments the filter is `.+`, which means it must have a pre-release tag.  For all the other channels the filter is `^$`, which means no pre-release tag allowed.

![](channel-prereleasetag.png)

We will get into the build server configuration a bit later.

You will notice any tenant can use the default channel.  Does this mean each tenant can deploy to production?  No.  When you link a tenant to a project, you tell Octopus the environments the tenant can deploy to.

![](tenant-overview-project-environments-highlighted.png)

## Tenants

The tenant tags are in place.  The lifecycles have been configured.  The projects have been configured.  The necessary infrastructure changes have been made.  It is finally time to create some tenants.  

For each tenant, I am going to assign a tenant tag.

![](tenant-with-tag.png)

Then I am going to make the connect my projects.  Because this is a Person `Tenant Type` I am only allowing deployments to the `Development` and `Testing` environments.

![](tenant-connect-project.png)

If you recall, the `Global` variable set has a variable template.  Now that the projects are connected I will need to populate that variable template by going to variables and then selecting `Common Templates`.

![](tenant-common-variable.png)

When it is all said and done, I created two tenants for the release `Tenant Type`.

![](release-tenants.png)

Three tenants for the team `Tenant Type`.  We name our teams pod-[Octopus Type].

![](team-tenants.png)

And finally, several tenants assigned the person `Tenant Type`.

![](person-tenants.png)

The tenant screen can filter by `Tenant Type`.  Filtering by `Tenant Type` makes it easier to find a specific tenant.  As well as taking screenshots after the fact.  Which I totally did.

Now the project overview screen will show all the tenants.  

![](databasedeployments-with-tenants.png)

I can deploy to a specific tenant by selecting a release.

![](projectoverview-select-release.png)

When you get to the deploy screen, you can choose to deploy to a specific tenant or tenants by tenant tag.  If you look at the bottom of the screenshot, it will show you all the tenants who will be deployed to.

![](tenants-deploy-screen.png)

The project overview screen also provides the ability to group tenants by tenant tag.  When we group by tenant type the screen will group all the tenants by their tenant type.

![](projectoverview-groupby.png)

## Build Server

I will be using TeamCity for this example.  The same core concept will apply for your build server of choice.

At the start of my build, I am running a PowerShell script to set various output parameters.

![](teamcity-build-process.png)

In my case, the PowerShell script is setting these three parameters.

![](teamcity-output-parameters.png)

Those parameters are then used in the create release step.

![](teamcity-create-octopus-release.png)

The PowerShell being called isn't anything super complex.  Check the branch name against a list of rules and go from there.  At the end of the script, it uses TeamCity specific formatting to set the output variables.

```PS
Param(    
    [string]$currentBranch,
    [string]$buildNumber,
    [string]$defaultMajorVersion,
    [string]$featureBranchVersion
)

Write-Host "BuildNumber: $buildNumber"
Write-Host "DefaultMajorVersion: $defaultMajorVersion"
Write-Host "FeatureBranchVersion: $featureBranchVersion"

$channelName = "Default"
$releaseVersion = "$defaultMajorVersion.$buildNumber"
$environmentName = "Development"
$tenantName = "Main"

Write-Host "The branch that is building is: $currentBranch"

if ($currentBranch -like "refs/heads/hotfix*") {    
    Write-Host "Hotfix Branch Detected, deploying to staging and using the hotfix channel"
    $channelName = "Hotfix"    
    $releaseVersion = "$defaultMajorVersion.$buildNumber"
    $environmentName = "Staging"
    $tenantName = "Hotfix"
}
elseif ($currentBranch -like "refs/heads/team-*") {    
    Write-Host "Team branch detected, deploying to a team tenant"
    $channelName = "Team and Developer"
    $environmentName = "Development"
    
    $preReleaseTag = $currentBranch.replace("refs/heads/team-", "").replace(" ", "").replace("/", "-")
    $releaseVersion = "$featureBranchVersion.$buildNumber-$preReleaseTag"
    
    $teamName = $currentBranch.replace("refs/heads/team-", "")
    $teamName = $teamName.substring(0, $teamName.indexOf("/"))
    
    $tenantName = $teamName
}
elseif ($currentBranch -like "refs/heads/developer-*") {    
    Write-Host "Dev branch detected, deploying to the team or team tenant"
    $channelName = "Team and Developer"
    $environmentName = "Development"
    
    $preReleaseTag = $currentBranch.replace("refs/heads/developer-", "").replace(" ", "").replace("/", "-")
    $releaseVersion = "$featureBranchVersion.$buildNumber-$preReleaseTag"
    
    $devName = $currentBranch.replace("refs/heads/developer-", "")
    $devName = $devName.substring(0, $devName.indexOf("/")).replace("-", " ")
    
    $tenantName = $devName
}
else{
    Write-Host "Master or release branch detected, using the defaults"
    $channelName = "Default"
    $releaseVersion = "$defaultMajorVersion.$buildNumber"
    $environmentName = "Development"
    $tenantName = "Main"
}

Write-Host "Tenant Name: $tenantName"
Write-Host "Environment Name: $environmentName"
Write-Host "Release Version: $releaseVersion"
Write-Host "Channel Name: $channelName"

"##teamcity[setParameter name='env.octopusChannel' value='$channelName']"
"##teamcity[setParameter name='env.octopusVersion' value='$releaseVersion']"
"##teamcity[setParameter name='env.octopusEnvironment' value='$environmentName']"
"##teamcity[setParameter name='env.octopusTenant' value='$tenantName']"
```

The final piece is telling your build server, in my case TeamCity, to monitor all branches, or to monitor branches which match your requirements.

![](teamcity-monitor-all-branches.png)

## Seeing it in action

Let's take a look at this in action.  I created a test branch for my developer and pushed it up to the server.

![](git-test-commit.png)

TeamCity picked up the new branch.

![](teamcity-pending-build.png)

The log indicated it found everything successfully.

![](teamcity-script-log.png)

And the deployment is kicked off for the tenant.

![](octopus-deploy-building.png)

I also have a specific package for that branch which can then be shared with other developers.

![](octopus-deploy-developer-package.png)

So if Derek wanted to bring it over, we would first change the release to my release number.  Then we would click on the deploy button.

![](octopusdeploy-derek-bringing-in-release.png)

## Conclusion

It takes a bit of setup, but it is possible to configure Octopus Deploy, so each team and each developer have their own sandbox to deploy to.  Having a separate sandbox will allow teams and developers to get feedback on in-progress work without having to step all over the main code base.

One final thought, I included a hotfix tenant.  You don't have to include that.  I did that because the customer I was talking to had a separate "hotfix" subdomain in their staging environment.  It made sense for them.  It might not make sense for you.  I would encourage you to test this out for yourself and tweak the process to meet your needs.