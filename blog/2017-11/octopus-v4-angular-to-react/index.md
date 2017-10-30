---
title: Octopus Deploy 4.0 - Moving from Angular to React
description: One of the biggest changes in Octopus Deploy 4.0 is a complete portal rewrite in React. 
author: robert.erez@octopus.com
visibility: private
tags:
 - New Releases
 - Portal
---

## Decision Points ##
### A Maturing Codebase ###
The front end code base that lives inside the `3.x` versions of Octopus Deploy started it's life back our [`2.0` move to Angular](https://octopus.com/blog/2.0). In 2013 Angular 1.2 was all the rage, dependencies were managed with bower, built with gulp and [Thrift Shop](https://www.youtube.com/watch?v=QK8mJJJvaes) was killing it on the airwaves. Since this initial release, the portal has been upgraded many times however it had still started to feel the strain of a product that had grown immensely, both in terms of functionality and number of active developers. Managing a small code base with 3 developers is much simpler than a large one with over 30 different people and like any code base, the existing architecture had become bloated, inconsistent and outdated. It roughly followed the old approach to structring "mvc-like" applications with a directory for the controllers, one for the views and a seperate one for the directives
![Create Deployment Page - v3 design](old-directory-structure.png "width=500")

This obviously falls apart fairly quickly once you have more than a couple of UI elements, who's constituent parts become split up over several directories. Through the newer pattern to co-locate files that make up a single component has since come, our existing app the old pattern reigns supreme. Maybe manageable back when Octopus had a simpler feature offering, but not so much now that there is so much more to support and control from the front-end. Organizational issues like these have no one to blame but ourseleves, but there are always issue like this that grow out of an evolving codebase.

### Performance Considerations ###
Scale had also become a concern as larger and larger customers began to notice performance issues in the portal when dealing with thousands upon thousands of machines, projects or steps and a large part of this is due to how Angular (pre 2.0) handles and renders state. As an experiment we ended up replacing some parts of our Angular code with small islands of React, hoping to leverage the benefits of a virtual dom, and removal of digest cycles. The results were clear. Rendering a screen of a few thousand tenants went from 20 seconds in Angular, to 2 seconds in React. While part of this may have been due to a much needed cleanup of our existing Angular code, it did show that some real improvements could be made if we rethought our existing codebase.

## Still... why React? ##
With the Angular 1.4 version of our portal clearly outgrowing itself it was decided that to better scale for the future, both in terms of ease of development and performance for the end user, rebuilding the front end was critical. Going from Angular 1.x to 2.0 would be seen as almost a full rewrite so what better opportunity to evaluate what other options were available. 

React had already displayed itself to be a clear contender. Buy utilizing the Virtual DOM (not to be confused with the [Shadow DOM](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Shadow_DOM), a browser native feature) React is able to reduce document rendering to only those instances where something has changed. When dealing with thousands machines as some of our users do, any improvement to the UI is noticeable. The observer pattern as provided with Angular through the digest cycle only exacerbates the problem. Comparing the state of all our components after any change just proved to be far too expensive.

Ultimately no-one develops in a vaccume (and by "vaccuum" I mean an environment that lacks StackOverflow as opposed to air) so the community engagement was a critical factor in our decision. The React ecosystem comes with a huge community of active contributors which in comparison with Angular 2.0, React won hands down. React has grown from just a simple front-end website framework, and is now available as a tool for building "native" browser apps (used in our mini IOS app [OctoWatch](https://itunes.apple.com/us/app/octowatch/id1232940032?mt=8)) or server-side rendering of websites.


## A New Hope ##
So having picked React as the core engine underlying the new portal, lets take a look at how we ended up using it to build the brand new portal.

### No Redux ###
From my experience, it seems as though lot of developers decide they want to learn React and upon reading a few tutorials are convinced that they need Redux and so start their new project with something like:
```
npm install --save react redux react-redux
```
There appears to be a bit of an assumption through parts of the community that you can't (or shouldn't) build a React app without managing all your state in Redux. Dan Abramov, one of the authors of Redux said it best in his blogpost ["You might not need Redux"](https://medium.com/@dan_abramov/you-might-not-need-redux-be46360cf367)

> However, if you’re just learning React, don’t make Redux your first choice.
Instead learn to think in React. Come back to Redux if you find a real need for it, or if you want to try something new. But approach it with caution, just like you do with any highly opinionated tool.

and later in response to a [twitter thread.
![Dan Abramov on Redux](dan-abramov-on-redux.png "width=500")](https://twitter.com/dan_abramov/status/725089243836588032?lang=en)

With our initial attempts at building the portal we started with the intention to use Redux as much as possible to maintain state across the project. Our aprehension started when we saw the amount of boiler plate required to perform the most mundane tasks. Our further concern was around the idea of storing so much data into a single, static (for all intents & purposes) state. The dashboards for some of our users with huge deployment dashboards can be several megabytes in size. Ensuring that this state is cleaned up at the right time involves dispatching events at the right time during the component lifecycle however this feels like a leaky abstraction, requiring the components to know something about the storage mechanism to know when it needs to be cleaned and when it can be left. Ultimately the context of this data only makes sense on the dashboard component.

Knowing the right place in the object graph also requires knowing something about the context of the application as a whole. When I retrieve the `Project` resource, should I put it just on the root level at "Project"? What about if another component suddenly needs another project for a different purpose, or if we decide that there could now be multiple Projects loaded concurrently for different parts of the screen? This questions all have answers in the form of conventions, rules or libraries, but their presence gave us pause for thought as for what we were doing with Redux in the first place.

We responded by stripping back our usage of Redux and only using them where the existing component state mechanism no longer made sense or was impractical. Yes the sub heading is a bit of click bait, we still use Redux however we only bring in state on a case-by-case bases when it makes sense. Parts of our app that need to communicate or deal with non-localized state made sense to live in the non-localized state management that Redux provides. Contrary to some initial concerns, we found this to be no worse off a development and debugging experience than what the "whole app state in Redux" approach purports to be. The only downside is that this can make testing that little bit more complex since the data isn't injected through Redux but loaded by the component itself during the `componentDidMount` lifecycle phase.

The key lesson for this is to understand your problems and limitations before looking for a solution, and to use that solution only where it makes sense, not just because everyone else is doing it.

[xx](https://daveceddia.com/what-does-redux-do/)

### Javascript With Types ###

Very useful on any team with more than 2 developers. Used with TSLint

## Going Forward ##