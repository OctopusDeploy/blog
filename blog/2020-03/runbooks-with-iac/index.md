---
title: Using Infrastructure as Code with Octopus Runbooks 
description: This post will discuss how to leverage Octopus Runbook with Infrastructure as Code
author: bob.walker
visibility: private
published: 2020-03-31
metaImage: 
bannerImage: 
tags:
 - Product
 - Runbooks
---

In preparation for the [2020.1 Release Webinar](https://www.youtube.com/watch?v=M5ejzd8KdbQ&feature=youtu.be), I needed to spin up and down the infrastructure for my demos.  Before runbooks, I would've had a separate "Infrastructure as Code" project to handle this.  I didn't want to do that again, runbooks were built with this scenario in mind.  The webinar prep allowed me to port that process over to runbooks.  Along the way, I had to adjust how I approached Infrastructure as Code in Octopus Deploy. This post walks through some of the lessons I learned in porting an existing Infrastructure as Code process into the new runbooks feature in Octopus Deploy.

!toc

## Brief Intro to Infrastructure as Code 

I want to ensure everyone is on the same page when I use the term Infrastructure as Code or IaC.  Infrastructure as Code is modeling the desired infrastructure in a file, typically written in YAML, JSON, or Hashcorp Language (HCL).  Cloud providers provide a mechanism that ingests that file and provisions your requested infrastructure.  Each cloud provider has its own infrastructure definitions:

- [AWS CloudFormation](https://aws.amazon.com/cloudformation/)
- [Azure ARM Templates](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/overview)
- [GCP Deployment Manager](https://cloud.google.com/deployment-manager/docs)

Using the tooling provided by the cloud providers has its advantages.  Typically a UI is created to aid in debugging.  The process is kicked off by a CLI, but when something goes wrong, you can splunk through the logs in the UI and find the root cause.  Also, to aid in the learning curve, they will either provide samples or allow you to define the desired infrastructure in the UI and export it to a file. Incidentally, this is how I create ARM templates.

The downside to using the provided tooling is the risk of lock-in.  The terminology for AWS Cloudformation doesn't translate over to Azure ARM Templates.  This is where third party tooling such as [Hashcorp's Terraform](https://www.terraform.io/) is useful.  They provide a common framework to be used across all cloud providers.  

## Infrastructure as Code before Runbooks

Before runbooks, my Infrastructure as Code process was a normal deployment process.  I wasn't ever "deploying" anything. Instead, I was running some scripts against a cloud provider.  My goal with my process was simple, a single project to tear up and down my infrastructure.  This led to some quirks.

The first quirk is I had to create a lifecycle.  That was because it was a deployment process.  For example, I have two environments, `Test` and `Production`.  However, I needed to have the ability to tear everything down.  I didn't want to tear down `Test` when I deployed to `Production`.  I also wanted to be able to tear down `Production`.  That need led to my second quirk.  I needed another environment called `Teardown`.

![](octopus_lifecycle_before_runbooks.png)

My deployment process was a bit complex.  It had steps which were either:

- Scoped to any environment but `Teardown`
- Scoped to `Teardown`

![](octopus_deployment_process_prior_to_runbooks.png)

You will notice in my lifecycle `Test` and `Production` are optional.  This brings me to my next quirk.  Configuring IaC for the first time, be it AWS CloudFormation, Azure ARM Templates, GCP Deployment Manager, or Hashicorp's Terraform, has a lot trial and error.  Typically I get the infrastructure created after one or two tries.  It is everything after which causes headaches.  A VM is created, but the bootstrap script missed something.  Properly testing the fix requires tearing down and spinning up the infrastructure.  Sometimes the error happened in `Test` and other times `Production`.  I needed a way to get to `Teardown` if the deployment actually failed.  Hence, `Test` and `Production` are optional.

## AWS Region Specific Settings

For this article, I am going to be using AWS CloudFormation to spin up a Ubuntu VM, install a tentacle on that VM, and register that tentacle with Octopus Deploy.  In choosing AWS, I stumbled upon another quirk.  Each region is segregated.  For the most part, resources cannot be shared between regions.  This includes:

- SSH Keys for VMs
- AMI Images
- Subnets
- Security Groups

That is not an all-inclusive list.  I _did not_ want to have an environment per region.  That scales poorly.  But I needed a way to have region-specific settings.  Time to leverage [Octopus Deploy's multi-tenancy](https://octopus.com/docs/deployment-patterns/multi-tenant-deployments) functionality.  The phrase `Tenants` was chosen for a reason.  A tenant can be

- A customer for a SaaS application
- A sandbox per developer
- A data center
- An AWS Region

Obviously, I picked the last option.

![](octopus_deploy_aws_tenant_regions.png)

One of my favorite features with tenants is each tenant can have a unique variable value per environment.  It makes sense for database connection strings.  In my IaC example, it would've led to a lot of duplicate values.  I don't want to have to define the same AMI on `Test`, `Production`, and `Teardown`.  Thankfully it is possible to configure tenant-specific variables to be used across all environments.  

### Option 1 - Leverage Tenant Tag Sets

First, I created a Tenant Tag Set called `AWS Region`.

![](octopus_tenant_tag_aws_region_set.png)

I did this because tenant tags can be scoped to a variable.  You can access that variable scoping by clicking on the `Open Editor` link.

![](octopus_open_scope_editor.png)

That will open up a modal window, which will allow you to select the tenant tag.

![](octopus_variable_scope_modal_window.png)

I created a variable set called `AWS`, which I populated with variables I'd need for my CloudFormation template.

![](octopus_aws_variable_set.png)

### Option 2 - Leverage Variable Set Variable Templates

I went with option 1 because it is easier to create screenshots for and demo to new users.  I've found most people "get it" when they see the above screenshot.  It's also useful in showing the differences in each AWS region.  However, it requires duplicating the tenant names as tenant tags.  

Another option is to leverage the Variable Set Variable Templates.  In variable sets, there is a `Variable Templates` tab.  First, add a variable template.

![](octopus_variable_set_variable_templates.png)

On the tenant variable screen, you'll see a `Common Templates` tab.  That variable template you added to the variable set will appear.  Fun fact, the California region gave me the most heartache.  That is why it has a an image of Arnold screaming California.  

![](octopus_tenants_common_variables.png)

:warning
The tenant must be connected to a project that is connected to the variable set.  
:

If I were setting up Octopus Deploy in an enterprise setting, I would go with this option.

## Porting The Deployment Process to Runbooks

I quickly learned I shouldn't do a straight port from a deployment process to runbooks.  I needed to change my thinking on how I configured my IaC process.  As stated earlier, I wanted a single process to spin up and down my infrastructure.  My "Infrastructure as Code" project's process reflected that goal.  However, that single project goal caused me to make some suboptimal configurations.  

Those suboptimal configurations were:

- The entire concept of a `Teardown` environment and the subsequent lifecycle.
- Having to scope a step to either `Teardown` or run in every environment but `Teardown`.  In other words, I had two processes jammed into one.  
- That `Teardown` environment required me to write scripts to tear down `Test` and `Production` at the same time.  
- Because I needed to tear down everything at the same time, I had to have complex scoping of variables.  

Let's take a moment to discuss the variable scoping.  That does an excellent job of shining a light on multiple problems with my process.  In `Test` and `Production`, the variable template for the VM name followed this format `[Application Name]-[Component]-[Environnment Abbreviation]`.  For example, `todo-web-t` for the web server for the To-Do application.  The `Teardown` the variable didn't include the environment abbreviation.  It was just `[Application Name]-[Component]`.  The `Teardown` process needed to deregister VMs for all environments.  That deregister script found and removed all targets that started with `[Application Name]-[Component]`.  That is a simple example.  Adding regions got more complex.  It took quite a bit of time to come up with a variable template that didn't result in accidentally deleting the wrong target. 

![](octopus_complex_scoping_variable_names.png)

### Split The Process

The first change I made was I split my IaC process into two runbooks.  One runbook spun up the infrastructure, while the other deleted the infrastructure.  

![](octopus_runbook_list.png)

Because of that split, I no longer needed the `Teardown` environment. This, in turn, simplified the variables.  I also learned I could scope variables to specific runbooks.  This meant I wasn't passing in unused variables to my deployments.  In other words, I could have simple variables, and they were only used for specific runbooks?  Sold!

![](octopus_scope_variables_to_runbook.png)

The process also became easier to wrap my head around.  I wasn't trying to smush together two processes into one.

![](octopus_runbook_process.png)

### Tenants

I still wanted to keep my tenant per region configuration.  However, my sample application isn't multi-tenant.  My application is a small .NET core web application running on NGINX with a NoSQL backend.  It doesn't require anything other than to be split across some regions.  Doing something like Canary deployments or Blue/Green deployments is overkill.

What I wanted was:

- When I deploy to test, deploy to the servers in the `Ohio` region.
- When I deploy to production, deploy to all servers across `Ohio`, `California`, and `Oregon` regions using a rolling deployment.

![](octopus_deploy_sample_deployment_process.png)

This is when I found out it is possible for a project to not allow tenanted deployments BUT a runbook can require them.  In my project settings I set it to disable tenanted deployments.

![](octopus_project_disable_tenant_deployments.png)

However, I set each runbook to require a tenant.

![](octopus_runbook_require_tenant.png)

Because runbooks don't have a lifecycle the overview screen looks a little different than the usual project overview screen you might be used to.

![](octopus_runbook_overview.png)

### Spinning up and down infrastructure on a schedule

VMs cost money.  For this example I don't need the VMs running 24/7.  I'd rather have them run during workdays only.  You might have a similar requirement for your testing environments.  If nothing is going to use the VMs, why keep paying money to have them running?

This led me to a slight problem.  The scheduled trigger for runbooks allows me to select 1 to N number of environments.  

![](octopus_runbook_trigger.png)

The trigger will kick-off two tasks, one for each of my specified environments.  With two environments, that isn't that big of a deal.  But I ran into a wall when I was configuring another example, and I wanted four environments.  The number of concurrent tasks running would exponentially grow.  I didn't want to overload my task queue with spinning up and down my infrastructure.  My first thought was I could schedule the triggers 15 minutes apart.  For example:

- Project A spin up `Development` infrastructure: 6:00 AM
- Project A spin up `Test` infrastructure: 6:15 AM
- Project B spin up `Test` infrastructure: 6:30 AM
- Project A spin up `Staging` infrastructure: 6:45 AM
- Project A spin up `Production` infrastructure: 7:00 AM
- Project B spin up `Production` infrastructure: 7:15 AM

That doesn't scale all that well.  Especially when I add more projects.  And sometimes it takes 5 minutes to spin up the infrastructure.  Sometimes it takes 20 minutes.  I didn't want to waste time waiting, but at the same time avoid overloading the task queue.

What I wanted was to create a runbook that would spin up all my infrastructure in an orderly fashion.  If you are familiar with JQuery promises, essentially I wanted to chain my promises.  Run one runbook at a time, and when it was done, start on the next one.  

You know what's good at waiting for something to finish and then kicking off the next step?  A runbook process.  I needed a way to have a runbook kick off another runbook.  But there is no built-in step to do that.  So I wrote this [step template](https://library.octopus.com/step-templates/0444b0b3-088e-4689-b755-112d1360ffe3/actiontemplate-run-octopus-deploy-runbook).  Now I can have a runbook spin up and down my infrastructure in an orderly fashion.  I called that runbook `Unleash the Kraken`.

![](octopus_runbook_which_calls_runbook.png)

I have two scheduled triggers, one to spin up everything, and another to tear everything down.  You'll notice the teardown trigger runs every day while the spin up only runs on the weekends.  That is in case any infrastructure is spun up on the weekends.  

![](octopus_runbooks_scheduled_trigger.png)

## Conclusion

There are a few changes to go from spinning up infrastructure using a deployment process to a runbook.  All in all, I am happy with making those changes.  Everything feels a lot cleaner.  

My absolute favorite part about runbooks is not having to create a release for each run.  To get my CloudFormation template and bootstrap script right, I had to do a lot of runs before landing on something which worked.  Not having to do the release dance was very nice.

This sample is available on our [samples instance](https://samples.octopus.app/app#/Spaces-102/projects/to-do-linux/operations/runbooks) for you to poke around on.  You will need to sign in as a guest.  Don't worry about breaking anything, the guest user has read-only permissions.