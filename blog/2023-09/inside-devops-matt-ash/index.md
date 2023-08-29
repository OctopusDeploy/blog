---
title: Inside DevOps with Matt Ash
description: A series where we share lessons learned from those on the frontlines of DevOps. This post features Matt Ash, Staff DevOps Engineer at a leading HR software platform provider.
author: joanna.wyganowska@octopus.com
visibility: public
published: 2023-09-18-1400
metaImage: 
bannerImage: 
bannerImageAlt: Photo of Matt Ash
isFeatured: false
tags: 
  - DevOps
  - Inside DevOps
---

This post is the next in our [Inside DevOps series](https://octopus.com/blog/tag/Inside%20DevOps), where we share lessons learned from those on the frontlines of DevOps.
  
Hear from Matt Ash, Staff DevOps Engineer at a leading HR software platform provider.

**How did your DevOps journey start?**

*Matt*: About 10 years ago, I was an Application Development Team Leader, and the company struggled with automated solutions. Our web application build and deployment process was severely lacking. We trialed Octopus Deploy and a handful of CI tools, and ultimately decided on TeamCity and Octopus. 

Through that effort, I discovered a passion for automation. A few years later, I transitioned from application development to full-time DevOps engineering and haven’t looked back. 

Today, with Octopus automating deployments, my DevOps team supports over 900 software engineers working on multiple applications. We set the patterns to follow, so as a company, we can deploy all our applications in a secure, reliable, and repeatable way, at scale.

**Your current role sounds like Platform Engineering to me, where you enable developers to operate their own product, within defined guardrails.**

*Matt*: Yes, that’s correct. Although we’re called a DevOps team, we improve developer experience and productivity by giving them self-service capabilities.

**What is DevOps to you? How do you define it?**

*Matt*: DevOps is automation. It’s simplistic, and all development is automation to some degree, but it’s focusing on removing manual operations in favor of automated processes and monitoring. 

Manual operations come with more risk and encourage only a few team members to hold knowledge. However, automated processes decentralize knowledge and make solutions more predictable, repeatable, and discoverable. This in turn leads to scalability.

**What’s the most challenging part of DevOps?**

*Matt*: Variability at scale. We have a small team, yet we service many software engineers who have a wide variety of products they build and deploy. Some use Windows, Linux, node, C#, Python. We need to be ready to help teams move forward. Sometimes that means crafting the best solution for the different needs of each team, without being a roadblock to their progress. You get to learn a lot in the process, but it does come with a bit of stress too. 

**What’s the biggest challenge Octopus has helped you with?**

*Matt*: You might notice a theme in my responses, but Octopus has helped us with enablement at scale. As a team of 4, we couldn’t service the number of developers we do without tools built with scalability in mind. 

Plus the Octopus Runbooks feature has been a fantastic way for us to automate a ton of work quickly, especially when integrated with a GitHub feed. We use Octopus Runbooks to automatically manage our GitHub users. We also use it to expose functionality for managing repositories that our engineers wouldn’t normally have, due to how we configure for security. These are all executed by runbooks that execute scripts pulled from GitHub as a feed source.


**What’s the most rewarding part of DevOps?**

*Matt*: Enablement at scale. When a small team can enable a large group of engineers to build and deliver many times each day or week without your intervention, you know you’re doing your job well. A well-engineered DevOps solution should render the team invisible. That includes both the happy path when deployments succeed, and how well you can enable teams to solve their own deployment issues when they fail.

**What are some DevOps best practices you and your organization have implemented?**

*Matt*: Our focus is on enablement. So when we’re confronted with a problem that needs a solution, we look for 2 things. We look for automation opportunities and ways to empower the users so that when all is said and done, they don’t need to come back to us. 

Another best practice is testability for your automation. I hear a lot of people talk about scripting as if it’s something different than coding, and I couldn’t disagree more. When I began my DevOps path, I took all the practices I learned in C# and translated them to PowerShell. 

Writing testable code (which is a skill in itself) and executing those tests before delivering is invaluable, whether it’s for a C# web application or a PowerShell module responsible for orchestrating deployments.

**What advice would you give those just starting their DevOps journey?**

*Matt*: Start small and find something in your day-to-day work that’s a repetitive task and automate it. For example, use Octopus to deploy a web application. You’ll see the benefits of the release automation right away. 

Another tip is to take script engineering seriously. Train not only in the fundamentals of writing it but also in how to test, package, and publish/deploy it. These things apply to not only your customers but your own solutions. Your future self will thank you.

**What DevOps book would you recommend reading?**

*Matt*: Most of my recent reading hasn’t been in the DevOps world, but I’d recommend [Accelerate: Building and Scaling High Performing Technology Organizations](https://octopus.com/devops/reading-list/#accelerate) by Nicole Forsgren PhD, Jez Humble, and Gene Kim.

**What's one thing about you that might surprise us?**

*Matt*: My wife and I love to travel – we’ve been to 6 of the 7 continents and hope to get to Antarctica in the next few years to make it 7 out of 7.

**Thank you for a very insightful conversation Matt; it’s much appreciated.**

:::hint
If you’d like to feature in our series, [Inside DevOps](https://octopus.com/blog/tag/Inside%20DevOps), please reach out to [Joanna on LinkedIn](https://www.linkedin.com/in/joannawyganowska/) to set up time for a quick chat.
:::