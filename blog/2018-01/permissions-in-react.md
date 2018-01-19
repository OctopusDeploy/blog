---
title: "Octopus React UI Permissions"
description: Inside look into the React UI permissions
author: nick.josevski@octopus.com
visibility: private
published: 2018-01-25
tags:
 - Permissions
 - React
---

As part of our big portal overhaul in Octopus 4.0, we made the move from Angular to React, some great background and details is covered in that that post. https://octopus.com/blog/octopus-v4-angular-to-react

An important part of the React code is the client-side permission checks. Octopus has a rich and large feature set. It also has a complex permission system to match those features that's evolved over time to support restricting users. For many customers permissions stay out of their way, but there are many customers who have opted into fine grained control of what teams and team members can see and do.


## In this post

!toc


## Background

The Angular portal had evolved with client-side code to check for permissions, the permissions developed out of a firewall style approach and the code essentially does asserts that the user has sufficient privilege to see something. At the same time the API asserts the same and will return 401 responses if the user lacks it.


## Porting to React

It was out of scope to have major changes to the way we work with permissions for the work associated with Octopus 4.0, so it would be a close one-to-one port of the logic in the Angular code. A side effect in the Angular code was if we didn't do the permission assert correctly and we tried to render some UI with data we didn't have, it would silently just not render, including silence in the console.

This feature of Angular doesn't exist in the React + TypeScript world, if we made a mistake by not catering for the right permission check inputs, the UI will crash as child components attempt to render and operate on data and properties expected to be there.


## UI Permission Code

The canonical example case is you're accessing a Project. You arrive on the overview screen and we show you releases for each associated an Environment, and when Tenanted deployments are enabled we also show the Tenant. For each component in the hierarchy of the page, we need to factor in the Project, Environment (and Tenant).

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

A benefit we gained in the 4.0 by having this code everywhere, is we can clearly tell you why you can't see something, that's the call out element and text.

```
    <PermissionCheck 
		permission={Permission.LifecycleView} 
		alternate={
			<Callout type={CalloutType.Information}>
				 The {Permission.LifecycleView} permission is required to view the deployments
			</Callout>
		  } />
```

## Problems

In the first few patch releases of 4.0 with the help of the early adopters we found the edge cases we had missed in alpha and beta testing. If you check the release notes half of first 11 patches in the 4.0.x branch had a permission related fix. For the customers impacted we provided workarounds or fast patches but often they were show stopper bugs. As with all major changes and especially with a brand new UI we expected to have bugs we missed as we shipped, and by 4.0.11 the UI was as stable as it was in version 3 when considering permission check impact on crashes.

These bugs fell into 3 categories:

 - Old mistake we ported over
 - A brand new mistake
 - The new UI meant permissions needed more thought
 
The first 2 were for the most part straight forward, on closer inspection let us find the bug in the old code and ensure it was right in the new one. The new mistakes were similar, in that we just made a mistake in how we did the permission check.

The third category, was the most time consuming, in a few cases it involved a specific set of permissions we hadn't encountered in our testing and how the different structure in displaying data and actions couldn't cope.

It's quite a substantial amount of code in the UI that repeats something we have to check server side for the API calls. I did a search for this blog post in our code, and we have 96 uses of `<PermissionCheck />` across 61 tsx files, and 137 `isAllowed()` in 45 tsx and ts files.git


## The Plan

We're undertaking some work to make configuring permissions better, to ensure our customers fall into the pit of success when trying to make use of permissions to manage what their users can see and do. This work lines up with some with the Spaces feature that was discussed on our 2018 road map. https://octopus.com/blog/roadmap-2018

First part of the plan is the permission code server-side cleaning that up ensuring we can leverage it better to help drive the UI. The objective there is to define what actions can be done on the resources that are returned from the API, so we can remove a large portion of the extra code in React.

If there were to be more Octopus related applications on other platforms such as this one: https://www.youtube.com/watch?v=mxKBxHNDLzc they also can benefit and leverage the API to drive their UI.


## Wrap up

To return to the focus of making Octopus API driven, we'll be removing as much of the client side permissions checking we can, in favor of driving the UI via the data returned from the API.

Stay tuned for more posts about Spaces and Permissions as we get stuck into it over the next few weeks. 

