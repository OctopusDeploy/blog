---
title: How To Build A PowerBI Report For Octopus Deploy
description: How to look at Deployment and Runbook History in Octopus Deploy to analyize. 
author: Jeff Taylor
visibility: private
published: 3020-06-20
metaImage: to-be-added-by-marketing
bannerImage: to-be-added-by-marketing
tags:
  - Engineering
  - CI/CD and DevOps
---

It recently, it dawned on me that I have been using Octopus Deploy for 5 years now. I do not remember how I first heard about Octopus Deploy, but before I started using Octopus Deploy, I had been automating deployments using RedGate SQL Toolbelt with SQL Compare with a combination of C# and batch files which I wrote to handle deployments.

I had a thought...I wonder if there are any metrics I can gather from the Octopus Deploy database for the past five years and create reports in PowerBI Desktop.

I started thinking about what I could determine, such as how much time and money has automation has saved my company and me. If I could place an actual number on this, perhaps I could help others in their fight for automation where the work for justification. In the past, it has been a challenge trying to convince management that automation was a good thing. The most significant issue I've encountered is the constant human error and weekends away from the family due to long deployment times.

Another thought I had was to highlight development issues, such as how many times we have to skip releases due to bugs not found in our lower environments or to justify more resources for more testing to management.

I love saving time, so the best tool I know of to put together some metrics and display the analysis of data is PowerBI Dashboard.

Below is a sneak peek of what I was able to put together, and we will go over how you can create a connection and pull data from your database. I've also included PowerBI Desktop templates for your Cloud and Local instance to help you get started quickly.

![PowerBI Report For Octopus Deploy](headlinerimage.png)

Let me summarize each section of the blog before we jump in
1. **Prerequisites:** This section covers the prerequisites for reporting from Octopus Deploy.
2. **Deployment History Charts:** This section covers each of the individual charts for deployments I created.
3. **Runbooks History Charts:** This section covers each of the individual runbook history charts I created.
4. **Reports:** This section covers the four initial reports and charts I created in the templates I provided.
5. **ROI** - Does Octopus Deploy really save me and my company time and money?
6. **Conclusion:** Wrap up!

## Prerequisites

### PowerBI Desktop - Setup
If you don't already have PowerBI Desktop installed go to my blog post for detailed instructions on how to install it https://blog.reviewmydb.com/2020/06/how-to-install-powerbi-desktop.html.

### Octopus Deploy - SQL Server
Learn how to connect to a local instance of Octopus Deploy by going to my blog post https://blog.reviewmydb.com/2020/06/how-to-connect-to-octopus-deploy-local.html.

# Deployment History Charts

The body of the post is where you share your hypothesis, how-to, or story.

### Subheadings

Use three ### to include H3 headings.

Use **Bold** text for UI labels, use single back-tics for `parameters` and `filepaths`, and three back-tics for code blocks:

```
Write-Host "Hello, World!"
```

Use the following to include images:

![Description of the image](/path/to/image.png "width=500")

## Conclusion

Close off the post by restating the main points of the post, share any closing thoughts, and invite feedback.

## Learn more

- [link](https://www.example.com/resource)
