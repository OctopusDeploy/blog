---
title: Deprecating Azure Management Certificates
description: We're planning to deprecate the use of Management Certificates for Azure Web App deployments. Here's what you need to know.
author: shannon.lewis@octopus.com
visibility: private
published: 2018-08-30
tags:
 - Azure
---

In this post I'm going to outline some changes that we are planning for `2018.9` related to Azure deployments.

For the most part this will be us moving some things around internally, but there is one change we're planning that will impact anyone still deploying Azure Web Apps using Management Certificates.

Let's start with a look at what's causing pain and what we're planning to change. Then I'll go through how you can prepare for these changes.

## Understanding The Problems

The primary issue we currently have with our Azure support is that to keep maintaining Cloud Services and also keeping trying to move forward is getting very complicated in the code base. Why? Because to support Cloud Services we have to keep using a very old set of Nuget packages in parallel with the newer set that provide support for the newer Azure features.

Related to this is the complexity of the code paths to support both Management Certificates and Service Principals for deploying Web Apps. We have to call completely different underlying APIs in these scenarios and the differences between how these APIs function has been the source of a number of issues/bugs.

## Our Plan

### Step 1: Isolate the Cloud Services code. 

We've been caught out by the complexity of referencing two different versions of the Azure libraries at the same time, so we're going to stop doing that. Well we're going to stop referencing them directly from the same project anyway.

We're going to use the extensibility features we revamped in Octopus 3.5 to include the Cloud Services implementation as an extension. Once this has been implemented it will be included and the Cloud Services extension will not be actively maintained beyond that.

This will have no impact on any current Octopus instances, they will continue to function as today.

### Step 2: Remove Management Certificate support for Azure Web Apps

This is the change that will more likely impact people. To allow us to move forward with support for things like Azure Storage steps and Azure Function deployments, as a couple of examples, we need to remove some of the things that are holding us back. One of the primary culprits in this space is the myriad of code paths created by supporting the two different account types and the two sets of APIs they have access to. So our plan is to remove support for Web App deployments using Management Certificates.

Another factor in this decision has been Microsoft's announcement about [deprecating the Service Management APIs](https://blogs.msdn.microsoft.com/appserviceteam/2018/03/12/deprecating-service-management-apis-support-for-azure-app-services/). So the writing is on the wall, this is approach is dated and its days are numbered anyway.

From an Octopus perspective, the change is hopefully a little less dramatic than it might first seem. We actually removed the ability to use Management Certificates for any new steps and any Azure deployment targets as of `2018.5`. So you'll only be impacted if you have deployment processes in an instance that has been upgraded from an earlier version and you are using Management Certificates to deploy Web Apps. Cloud Services can only be deployed using Management Certificates, so they will stay unchanged.

## What Do I Need To Do To Prepare?

If you are deploying Cloud Services or you're using a Service Principal to deploy your Web Apps, there is nothing you need to do. We're moving some things around behind the scenes to let us bring you bigger, better things in the future but you shouldn't see any obvious differences with this change. This counts equally for whether you are using cloud targets or still using steps with the account selected directly on the step.

If you are using Management Certificates for Web App deployments you will need to switch to using Service Principals or stay on a version earlier than `2018.9`. To do the conversion you have two options.

The first option is to edit your existing step and select a Service Principal account in the dropdown. 

The second option is to switch to using deployment targets. This is the recommended option, as using deployment targets is the only option supported when adding new step instances and is how all newer step types will work. To do this conversion you need to create the targets and assign them a role. You then edit the step to clear the account field and select "run on behalf of" the role for your target.

## Wrapping Up

We do try to maintain backward compatibility as much as possible and for as long as possible. Hopefully it's clear that this change is beneficial for everyone in the longer term, and we hope the impact in the short term will be minimal and only for a small number of customers.

If you have any concerns about this change please let us know below.