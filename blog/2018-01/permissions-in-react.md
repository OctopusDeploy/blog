---
title: "Octopus React UI Permissions"
description: Inside look into the React UI permissions
author: nick.josevski@octopus.com
visibility: public
published: 2018-01-25
metaImage: metaImage-react-permissions.png
bannerImage: blogImage-react-permissions.png
bannerImageAlt: role-based access control
tags:
 - Engineering
---

As part of our UI overhaul in Octopus 4.0, we made the move [from Angular to React](https://octopus.com/blog/octopus-v4-angular-to-react), some great background details are covered in that post. 

Octopus has an extensive feature set.  To accompany the wide range of features, there's a complex role-based access permission system that has evolved to support restricting what users can see and do. Many customers choose to avoid tuning permissions and operate as a set of administrators with full access. For many organizations, that's not suitable, and they have opted into fine-grained control of what teams and team members can see and do.

We half-jokingly, half-seriously, wanted to completely remove the fine-grained permissions and the large set of role-based groupings and push customers to a simplified read and write model inside Octopus. With some exceptions like providing security around sensitive data passwords, private keys, etc. 

But that would have enraged thousands of customers.

In this blog post, I would like to share why those of us who maintain and interact with the permission system like to dream of a world where permissions are far simpler in Octopus.

![role-based access control](blogImage-react-permissions.png)

## In This Post

!toc


## Access Permissions Background

There's a long history of how the current role-based access control system has evolved into what it is now. Initially, we modeled it like a firewall and attempted to block access to restricted items. Soon after, it evolved into a set of actions a user can do which were scoped to a set of projects, environments, and tenants. When a user attempts to perform an action, we calculate if they have the suitable permission for the resource they're attempting to act on, e.g., Deploy Project ABC to Staging. 

Did I lose you? I hope not. I'll give more details on the role-based access control section later in the post.

The key takeaway is the current system in Octopus has a significant learning curve, has nuances and most importantly is in wide usage with a lot of investment from our customers.

Many customers have spent significant effort fine-tuning permissions for their large user bases. This means there is significant value in what the permission system is currently capable of and it also means we can't drastically change it, or reduce it to read and write access as we like to dream.


Our dream of a simpler permission system comes from a desire to reduce maintenance on some aspects of the permission code base. Having ruled out removing it entirely, and any major disruption to what it can do, this left us with a safe and reasonable third option.

Make what it currently does easier to reason about, make the code better to maintain, and reduce duplicated code.


## From Angular UI to React UI

The Angular UI evolved with client-side code to check for permissions; the code essentially asserts that the user has sufficient privilege to see something. At the same time, the API asserts the same thing with its own code, and will return 401 responses if the user requests data they can't access or attempts to perform an action they can't.

It was out of scope to have major changes to the way we work with permissions as part of Octopus 4.0, so it was a close one-to-one port of the logic in the Angular code. The Angular code would fail silently and invisibly if we didn't write the permission assert correctly. This meant, it was more forgiving, it just wouldn't do anything in that case.

In the world of React and TypeScript, if we made a mistake by not catering for the right access permission check inputs, the UI would crash as child components attempted to render and operate on data and properties they expected to be there.

The true guard of the data and capabilities of Octopus is the API. There were no significant changes there in 4.0, and as a result, we knew we weren't going to expose information or allow users to perform actions they could not. The worst thing that would happen (and did) was the UI would break and become unusable. This meant we had to get it right and it was a lot of work in bug fixing.

## Patch, Patch, and Patch

In the first few patch releases of 4.0, and with the help of the early adopters, we found the edge cases we missed in alpha and beta testing. If you check the release notes, half of the first 11 patches in the 4.0.x branch had a permission related fix. For the customers impacted, we provided workarounds or fast patches, but often they were showstopper bugs. As with all major changes and especially with a brand new UI we expected to have bugs we missed as we shipped, and by 4.0.11 the UI was as stable as it was in version 3 when considering complex access permission combinations and catering for them in the UI.

React and TypeScript continued to pay off for us while we shipped patches, especially as we tweaked the layout to match access permissions for the cases we didn't cater for, giving us confidence in refactoring.

## The React UI Bugs

These bugs fell into 3 categories:

 - Old mistake we ported over
 - Brand new mistakes
 - The new UI meant permissions needed more thought
 
The first 2 were straightforward, on closer inspection and with customer reports, we could find the bug in the old code and ensure it was right in the new one. The new mistakes were similar, in that we just made a mistake in how we did the access permission check and reports form, customers were key there too.

The third category was the most time consuming, in a few cases it involved a specific set of user-roles and permissions (that we hadn't encountered in our testing) and how the different structure in displaying data and actions couldn't cope. This involved adjusting how and when API calls were made and the hierarchy of react components that caused the crash, based on that user role and scoping combination.


## Role-Based Access Control

To paint a full picture of the of the user-roles in Octopus and how they drive access permissions, we have a list of security permission grouped into user-roles that align with common ways to break up access for doing tasks in Octopus. If you're using our-built-in user-roles to control the security and you're scoping them simply, you are unlikely to hit the edge cases of the React UI permission code.

It's for the customers who have crafted very strict permission configurations for their users where the edge cases come up. Such customers have fine-tuned user-roles to minimize what users can do. At the time of writing this, we have 103 of these permission roles. Yes 103, and they vary in what they can be scoped to, from global (nothing) to Project, Project Group, Environment, and Tenant. This translates into lots of access permission permutations (say, that fast 3 times). There are also non-obvious connections between permissions that are required to display certain parts of the React UI, and even if you're working with access roles every day, each time you adjust the access permissions in a fine-grained way, it's lots of testing and adjusting until you get it right.

We'll be addressing this usability and configuration complexity in the near future.


## UI Access Permission Code

The canonical example is you're accessing a Project, you arrive on the overview screen, and we show you releases for each associated Environment, and when [Tenanted deployments](https://octopus.com/docs/deployment-patterns/multi-tenant-deployments) are enabled we also show the Tenant. For each component in the hierarchy of the page, we need to factor in the Project, Environment, and (possibly) the Tenant.

This is our wrapping React component, to check if you can deploy:

```
	<PermissionCheck
		permission={Permission.DeploymentCreate}
		project={projectId}
		environment={environmentId}
		tenant={tenantId}>
			<NavigationButton label="Deploy" href={deployUri} />
	</PermissionCheck>
```

A benefit we gained in the 4.0 UI by having this react access control component everywhere, is that we can clearly tell you why you can't see something, that's the call out element and text.

```
	<PermissionCheck 
		permission={Permission.LifecycleView} 
		alternate={
			<Callout type={CalloutType.Information}>
				The {Permission.LifecycleView} permission is required to view the deployments
			</Callout>
		}>
			...
	</PermissionCheck>
```

It's ok, once you get used to it, but it has significant room for error, the inputs are document ids and are strings, they could easily be applied to the wrong filter. Some better type safety could mitigate that. If that's out of the way the next major challenge is still there; actually writing that code and ensuring it's in the right place.

There's quite a substantial amount of code in the UI that repeats something we must check server side. I did a search for this blog post in our code, and we have 96 uses of `<PermissionCheck />` across 61 TSX files, and 137 `isAllowed()` in 45 React TSX and TypeScript files.

This leads to the plan...

## Going Forward

We're undertaking some work to make configuring the role-based access permissions better, to ensure our customers fall into the pit of success when trying to make use of permissions to manage what their users can see and do, and of course not break or cause a lot of new effort for existing customers. This work lines up with the Spaces feature that was discussed on our [2018 road map](https://octopus.com/blog/roadmap-2018). 

The first part of the plan is to clean up and make testing the server-side permission easier, making it more robust, and easier to leverage in driving the UI. The objective is to define what actions can be done on the resources that are returned from the API so that we can do away with the need for a large portion of the extra code in React. Less code there means more safety, less maintenance and more time for us to work on more important features.

Another beneficiary from permissions driven by the API would be other applications that leverage our API like the iOS [OctoWatch](https://itunes.apple.com/us/app/octowatch/id1232940032?mt=8) if you're curious about it, here's one of our [TL;DR videos](https://www.youtube.com/watch?v=mxKBxHNDLzc)) covering it's creation and development using React Native.


## Wrap Up

To return to the focus of making Octopus API driven, we'll be removing as much of the client side permissions checking as we can, in favor of driving the UI via the data returned from the API.

Stay tuned for more posts about Spaces and Permissions as we get stuck into it over the next few weeks.

