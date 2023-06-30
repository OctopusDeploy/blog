---
title: Octopus Script Editor Uplift
description: A brief summary of the post, 170 characters max including spaces.
author: ellen.long@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - tag
---

See https://github.com/OctopusDeploy/blog/blob/master/tags.txt for a comprehensive list of tags,

Over the course of 2022, we ran a series of usability tests to evaluate the first deployment experience. Our aim was to gain valuable insights into the user journey and identify areas of improvement. One of the areas we identified was the script editor. Although functional, it had some usability issues and wasn’t occupying a prominent position within the script steps. We took this as an opportunity to give the script editor a fresh new look and uplift the user experience.


## Before
<img width="1042" alt="blogimage-scripteditorbefore-2023" src="https://github.com/OctopusDeploy/blog/assets/80728059/12dc26da-b0f8-4a35-bc04-8c9cd86e9538">


## After
<img width="1042" alt="blogimage-scripteditorafter-2023" src="https://github.com/OctopusDeploy/blog/assets/80728059/7d2149a7-b58f-49ed-b424-de58bb27ba04">


## What we changed

### Improved hierarchy and visual prominence
To enhance the visual hierarchy of the script editor, we moved it higher in the Run a Script step and inverted it to dark theme. This styling treatment gives the script editor prominence, making it the star of script steps.

![blogimage-scriptplacement-2023](https://github.com/OctopusDeploy/blog/assets/80728059/4becc8a8-d190-4310-9a97-11b300ee1872)


### Centralized and easy to use actions
To make the script editor easier to use, we introduced a toolbar to house its actions: language selector, copy to clipboard, insert variable and full screen buttons. Before, functionality was scattered and inconsistent across inline and full screen mode, limiting what you could do in either mode. By introducing this toolbar, all actions are available to you regardless of the mode you prefer to read and edit your code in, and you no longer need to dart your cursor around the screen to click on them.

In addition to centralizing the actions, we looked at how they were presented. Previously, action buttons were icons, which in our observations of users, made it difficult to discern their purpose and functionality. This was especially troublesome for the insert variable action as its ambiguous icon was also inconspicuously placed. Our solution was to add button labels. Now, you can understand what a button does at a glance.

![blogimage-scripttoolbar-2023](https://github.com/OctopusDeploy/blog/assets/80728059/e25279ea-7d3c-4afb-8fcb-77ba72705d99)


### Other notable changes
Alongside the major enhancements, we also implemented several smaller yet impactful quality of life updates. Notably, we’ve introduced placeholder “hello world” text that changes to reflect the syntax of the language you’ve selected, and to enhance your coding experience the editor now conveniently expands when you click into it to give you more room to read and edit your code inline.

![bloggif-scriptexpand-2023](https://github.com/OctopusDeploy/blog/assets/80728059/3e05afd0-8d7b-44e6-9c61-15ed2f7b9ecd)


## Expanding the scope
Our initial plan was to only roll these changes out to script editors in steps, but once we saw the benefits of the changes we decided to ship the update to all instances of the editor in Octopus. We did this by componentizing the script editor to make actions within the toolbar togglable. Depending on where the script editor is, it may need more or less functionality, and we can toggle parts off depending on where it’s used. 

For example, the code editor in script steps has the full toolbar, whereas the code editor for variables only requires the language selector and copy to clipboard actions.

## What’s next?

The new script editor provides an enhanced experience that makes it easier for you to read and edit your code in the Octopus product. 

We’d love to hear your feedback on this feature update and we want to know how you use the script editor. Do you prefer to edit code directly in Octopus, or copy and paste it in from somewhere else? Let us know below!

<span><a class="btn btn-success" href="https://octopusdeploy.typeform.com/to/bJfRWHyf" target="_blank">Share your feedback</a></span>

Happy deployments!
