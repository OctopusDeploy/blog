---
title: Octopus script editor uplift
description: Find out how we improved the script editor experience in Octopus Deploy.
author: ellen.long@octopus.com
visibility: public
published: 2023-07-10-1400
metaImage: blogimage-scripteditoruplift-2023.png
bannerImage: blogimage-scripteditoruplift-2023.png
bannerImageAlt: Two vector people putting together and painting the new Octopus script editor UI.
isFeatured: false
tags: 
  - Product
---

During 2022, we ran a series of usability tests to evaluate our customers' first deployment experience. We aimed to gain valuable insights into the user journey and discover areas of improvement. 

One area we identified was the script editor. Although functional, there were usability issues and the script editor wasn’t in a prominent position in the script steps. We decided to give the script editor a fresh look and improve the user experience.

In this post, I explain what you can expect from the uplifted script editor.

## Before

Here you can see what the script editor looked like before the uplift.

![Screenshot of the previous script editor.](blogimage-scripteditorbefore-2023.png "width=500")


## After

Here you can see what the script editor looks like now.

![Screenshot of the new dark theme script editor.](blogimage-scripteditorafter-2023.png "width=500")

## Changes to the script editor

### Improved hierarchy and visual prominence

To enhance the visual hierarchy of the script editor, we moved it higher in the Run a Script step and inverted it to dark theme. This styling treatment gives the script editor prominence, making it the star of script steps.

![Before and after screenshots comparing placement of the script editor in the Run a Script deployment process step.](blogimage-scriptplacement-2023.png "width=500")


### Centralized and easy to use actions

To make the script editor easier to use, we introduced a toolbar to house its actions: 

- **Language selector**
- **Copy to clipboard**
- **Insert variable** 
- **Full screen**

Before, functionality was scattered and inconsistent across inline and full-screen modes. This limited what you could do in either mode. All actions are now available to you regardless of the mode you prefer to read and edit your code in. You no longer need to dart your cursor around the screen to click on actions.

We also looked at how we presented the actions. Previously, action buttons were icons. When observing our customers, we noticed they had trouble working out the purpose and functionality of actions. This was especially noticeable with the **Insert variable** action as its ambiguous icon was also tricky to find. Our solution was to add button labels, so you can understand what each button does at a glance.

![Screenshots showing the before and after placement of script editor actions.](blogimage-scripttoolbar-2023.png "width=500")

### Other notable changes to the script editor

We also implemented several smaller yet impactful quality-of-life updates. Notably, we introduced placeholder “hello world” text that changes to reflect the syntax of the language you select. And to enhance your coding experience, the editor now expands when you click into it, to give you more room to read and edit your code inline.

![Gif shows the script editor expands when it is clicked, and placeholder “Hello World” syntax changes when a different coding language is selected.](bloggif-scriptexpand-2023.gif "width=500")


## Expanding the scope

We initially only planned to update script editors in steps. But after we saw the benefits of the changes, we decided to ship the update to all instances of the editor in Octopus. 

We componentized the script editor to make actions in the toolbar togglable. Depending on where the script editor is, it may need more or less functionality, and we can toggle parts off depending on where it’s used. 

For example, the code editor in script steps has the full toolbar. The code editor for variables, however, only requires the **Language selector** and **Copy to clipboard** actions.

![Screenshots comparing two versions of the script editor's toolbar. The first with all the toolbar actions, the second shows less actions.](blogimage-scriptcomponentization-2023.png "width=500")

## Conclusion

The new script editor provides an improved experience. It's now easier for you to read and edit your code in the Octopus product. 

We’d love to hear your feedback on this update and to learn how you use the script editor. Do you prefer to edit code directly in Octopus, or copy and paste it in from somewhere else? Let us know in our feedback survey.

<span><a class="btn btn-success" href="https://octopusdeploy.typeform.com/to/bJfRWHyf">Share your feedback</a></span>

Happy deployments!