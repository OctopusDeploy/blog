---
title: Better release notes with templates and automatic generation
description: A look at release notes templates and automatic release notes generation in Octopus.
author: shannon.lewis@octopus.com
visibility: public
published: 2019-04-08
metaImage:
bannerImage:
tags:
 - Product
---

This post follows on from our **Octopus Deploy 2019.4** announcement about [Tracking Your Work From Code to Deployment](../metadata-and-work-items/index.md).

An important part of the tracking is visibility and traceability, and we’ve already seen in the previous post how the Jira integration can help with that.  In this post, we’re going to look at another new feature, [release notes templates and automatic release notes generation](https://octopus.com/docs/packaging-applications/build-servers/build-information).

This is a feature that came out of the Jira integration, but isn’t dependent on that integration and in itself can make managing your release notes simpler. So if you’re managing release notes in Octopus, please read on.

## Release changes and release notes templates

It became apparent during our development of the Jira integration that there was also real value to be gained in the releases and deployments themselves from the new package metadata we now have access to.

With this thought in mind, we set about adding new variables in the deployments, for accessing the release changes. The first thing we did with this was use it to create email step and use the variable to create the HTML for an email.

This was great. It felt really useful. But the output was only useful if you were receiving the email. What about in the Octopus portal, wouldn’t it be nice to see this information in there? Yes it would, so we did that.

In the first iteration, we rendered it in a fixed read-only control. This was great and it felt really useful, but it was read-only. Would it fit all of the use cases that people would come up with?

History told us no :) So we iterated with the focus on "How do we allow customization of the release notes layout?" The result is release notes templates.

A release notes template is defined in the project settings and an example might look like this:

```
Here are the notes for the packages
#{each package in Octopus.Release.Package}
- #{package.PackageId} #{package.Version}
#{each workItem in package.WorkItems}
    - [#{workItem.Description}](#{workItem.LinkUrl})
#{/each}
#{/each}
```

You can use any valid markdown, as you’ve always been able to with release notes, but now the variable substitution is applied as part of the release creation so you will see it in the portal immediately. 

![Release with package metadata](release-work-items.png)

Also note, if you edit a release, you will see the text that resulted from the create, not the original template content. You can use variable binding in the edits, and they will be applied on save.

## Deployment variables

As we mentioned above, the deployments have been extended to include "release changes." An important point about this is the **deployments will always aggregate release notes from the release(s)** into the release changes, even if there is no metadata and work items. In other words, **even if you aren’t using any of the other package metadata and work item functionality, you can still take advantage of the accumulated release notes during a deployment**.

For each release related to the deployment, the release changes include a version (the release version), the release notes (in markdown format), and a list of work items.

Like our example from earlier, a common use for this information is in an email step. Below is a sample email template, including a link back to the release, and the release notes reformatted from markdown to HTML:

```
<p>Here are the notes customized for email</p>
#{each change in Octopus.Deployment.Changes}
<strong><a href="(#{Octopus.Web.ServerUri}#{Octopus.Web.ReleaseLink}">#{change.Version}</a></strong></br>
#{change.ReleaseNotes | MarkdownToHtml}</br>
#{/each}
```

## Wrap up

We’ve had numerous requests for release notes enhancements, and we’re really excited to finally ship it. If you’re using release notes in Octopus or keen to, please give it a go and let us know what you think.
