---
title: Our React Journey
description: Sharing some thoughts and lessons learned from the last couple of years using React.
author: mark.siedle@octopus.com
visibility: private
bannerImage:
metaImage:
published: 2019-10-30
tags:
 - frontend
 - react
---

Back in 2017, we published a [blog post](/blog/2017-11/octopus-v4-angular-to-react.md) where we started our journey with React. In this post, we wanted to share a followup and tell you about some of the lessons we've been learning and observing along the way.

## The Good

### Type Safety

This is an obvious one, but for shared team code (and many PRs being merged every day), TypeScript has allowed us to code with more confidence.

### Prettier

As more developers started coming on board here at Octopus, it became apparent that more and more people were coding in different styles. This is something I'm sure all of us have come across at one point when working with others, but we found it was getting out of hand because people would feel inclined to start formatting code in PRs and this increased tensions internally, because people argued that should be a separate "formatting PR" so that it was clear what things were formatting vs business-logic changes.

Then we were arguing (happily debating?) about which _way_ to format TypeScript. Ctrl-Shift-F in VSCode, vs some engineers manually aligning things, vs engineers using WebStorm. It'd be slightly different each time and kept up coming up every week in PRs.

Prettier

### Knowledge Sharing

To help spread the word about common patterns to use/avoid, we started doing more knowledge sharing sessions internally. This helps everyone get on the same page about architectural decisions, traps to avoid etc.

## The Not So Good

### People Not Speaking Up

When engineers don't understand something, they'll tend to do one of two things. 1) Make lots of noise and tell you it's too complex, or 2) Stay quiet and just avoid having to work in that code.

Looking back over the years, there's been a clear mix of the two going on.

The problem with 1) is they can tell you it's too complex, but often they can't pinpoint exactly why. But this is great. This is something you can start to tease apart and say _"Why do you find this confusing?"_ and start going down that road together.

The problem with 2) is, well, you don't get any feedback at all. To combat this, we're aiming to introduce internal surveys to our engineers. These can be anonymous, but we want to start capturing a Net Promoter Score of different aspects of our codebase to better understand where people are coming up against complexity. For quiet introverted engineers, we're hoping this gives them an outlet and will give us the key data we need to start increasing knowledge-sharing in given areas and analysing areas of our code people view as complex to see if there are some architectural improvements we can explore to make things easier.

### Inconsistencies Spreading Quickly Over Time

We had some feedback internally that people found Redux confusing. So we set off to try and make our Redux implementation easier for new developers to follow, and most importantly, try to better understand why people were finding it complex.

After a code-review, it quickly became apparently that we had some naming inconsistencies (which had grown to fairly epic proportions over the years).

Let's say you have a feature flag called `isHelpSidebarEnabled` that represents when the sidebar feature is enabled. E.g.

```
interface DispatchProps {
    isHelpSidebarEnabled: boolean;
}
```

We had inconsistent namings for these globally-injected interfaces all over the place:

`interface DispatchProps`
`interface ConnectedProps`
`interface InjectedProps`
`interface ReduxStateProps`

In some cases, we hadn't even declared a separate interface for the globally-injected props, so you'd just see global properties jammed into the component's main prop interface. Good luck trying to figure out where certain props were coming from, and if you were new to this, it was confusing trying to figure out whether these interfaces were named differently on purpose or not between different components.

So the first job was to audit the entire codebase for these inconsistencies and come up with something people could get some context from. We renamed everything consistently to `GlobalConnectedProps`.

Now when you stumble across this anywhere in our codebase, it tells you these props are connected from our global store (hey, it's a starting point at least).

Taking it a step further, we looked at the methods used to connect Redux to your component (in this case, a `DrawerWrapperLayout` component):

```
const mapStateToProps = (state: any) => { ... }
const mapDispatchToProps = (dispatch: any) => { ... }
const DrawerWrapperLayout = connect(
    mapStateToProps,
    mapDispatchToProps
)(DrawerWrapperLayoutInternal);
```

Couple of issues here...

There's no return types on those methods. Someone looking at this code for the first time can't connect-the-dots and realise that `mapStateToProps` provides your `GlobalConnectedProps`. So we had some typing ommissions going on.

Oh, and look, `any` used in a bunch of places instead of the actual types ><

Also, these function names don't tell you that this has anything to do with our global Redux store! Someone looking at this for the first time (or anyone inexperienced with Redux) would have no context about what these methods do or that they're in anyway related to our global state.

So we audited the codebase and refactored:

```
const mapGlobalStateToProps = (state: GlobalState): GlobalConnectedProps => { ... }
const mapGlobalActionDispatchersToProps = (dispatch: Dispatch<Action>): GlobalDispatchProps => { ... }
```

Now, you see `GlobalConnectedProps`, you see a method called `mapGlobalStateToProps` that returns the expected type and you can start to find your way around using the type system.

Not having the typing in place was one major cause of the confusion. There were other causes of complexity around Redux and selectors/reducers that we also smoothed out, but the point is, some consistent naming helped to make our codebase easier to reason about for everyone and can greatly improve the Developer Experience (DX).

The fact was, our foundation was inconsistent and those inconsistencies spread very quickly, causing longer-term damage to the confidence in our codebase internally.

### Complexities of TypeScript and Higher Order Components

We mentioned the use of `any` above, but this problem went much deeper, again from inconsistent foundational code in the initial port that then spread due to copy/pasta over the years.

TODO: markse - I think it's worth trying to summarise how we ended up just using `any` for things because the typing gets complicated around things like HOCs. Show that same example from Shaun's knowledge-sharing session today showing people what we're talking about and how much easier it is for devs to just go "screw it, any".
