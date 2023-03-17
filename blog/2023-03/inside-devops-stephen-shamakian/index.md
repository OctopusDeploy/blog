---
title: Inside DevOps with Stephen Shamakian
description: A series where we share lessons learned from those on the frontlines of DevOps. Our next post features Stephen Shamakian, DevOps Senior Principal Engineer.
author: joanna.wyganowska@octopus.com
visibility: public
published: 2023-03-20-1400
metaImage: blogimage-insidedevopsstephenshamakian-2022.png
bannerImage: blogimage-insidedevopsstephenshamakian-2022.png
bannerImageAlt: Photo Stephen Shamakian
isFeatured: false
tags: 
  - DevOps
  - Inside DevOps
---

This post is the next in our [Inside DevOps series](https://octopus.com/blog/tag/Inside%20DevOps), where we share lessons learned from those on the frontlines of DevOps.
  
Hear from [Stephen Shamakian](https://www.linkedin.com/in/stephenshamakian/), DevOps Senior Principal Engineer at a leading health solutions company.

**How did your DevOps journey start?**

*Stephen*:  I started in a web engineering role and found myself solving problems via automation. The projects I remember most were SSL cert renewals and IIS deployments.
 
At the time, the IIS deployment automation was a constant issue. I wanted to make our developers’ lives easier and keep them focused on what they love to do. We built something in-house, but I was looking for a more advanced system with a good UI and a powerful API for extensibility. 

That’s when I found Octopus Deploy, which sparked my interest in DevOps. From there, I branched out, but Octopus remains an integral part of this journey! Currently, we have ~13,000 projects and ~4,300 users of Octopus.

**And what does DevOps means to you?**

*Stephen*: It's not an original idea, but it’s the one that has resonated with me the most: DevOps isn't just one thing; it’s a combination of 3 things: 

- People
- Processes
- Tools

All 3 are equally important for the system to work well. 

For DevOps to work, you need buy-in from people so they embrace the cultural change that comes with the new tools and processes you’re introducing. You also need to define a CI/CD process and automate it. The third aspect is using the right tools. I view the tools as the vehicles that get you down the road. They’re important, as they help meld the people and process parts together.

**What are some DevOps best practices your organization has implemented?**

*Stephen*: We’re driving towards standardizing our DevOps pipeline enterprise-wide. We follow a build-once-deploy-many process. Before Octopus, we were building and deploying for every environment. This added a lot of intermittent failures. It caused audit concerns, and introduced potential risks to the process. We’ve fully moved away from that older, ineffective process. We’ve also moved to a change ticket-controlled production deployment process. All production deploys must have a scheduled change ticket assigned to them and a valid window to proceed. This lets us track and audit changes across the environment.

**What's the most challenging part of DevOps?**

*Stephen*: Well, going back to the 3 pillars that define DevOps for me, it’s a split between people and process. DevOps is very much a culture change in how development, operations, and even security work together. Even though DevOps aims to improve this, in many cases, these areas still function in silos. There are times when one area implements something that blocks another. And as a DevOps leader, you’re often in the middle trying to figure out the best path forward and an acceptable middle ground.

**And what's the most rewarding part of DevOps?**

*Stephen*: The automation, by far! Seeing something you worked very hard on pay off by enabling quick and reliable deployments for years to come is priceless! It’s like you’re breathing life into a process that never existed before, and it’s now functioning completely autonomously. Hence it has a life of its own. That’s super rewarding to experience in this role.

**What advice would you give folks just starting their DevOps journey?**

*Stephen*: This is a hard one as DevOps, for the most part, has 2 foundational paths. You can come at it from an infrastructure or a development angle. 

If you’re on the infrastructure path, I think the basics of system administration are a good start. The next step is viewing things with an automation mindset. For example, how can you script a task you perform manually? From there, learning scripting languages, like PowerShell and Python, is huge for any DevOps-related role. 

If you want to get into DevOps through the development path, then learning the basics of a language and building simple applications are foundational. But to take that a step further, how do the build mechanisms work to compile your code? How do you automate that? 

It’s a great time for beginners, though. There are training resources online, cloud providers for virtual labs, and of course, Octopus Deploy providing you with best practices for software deployments.

You also have to work on your soft skills. You need to be able to explain complex technical concepts in an understandable way, so you can get buy-in for proposed process changes. 

**Can you recommend a good DevOps book?**

*Stephen*: It’s been a while since I read it, but I loved [The Phoenix Project](https://octopus.com/devops/reading-list/#the-phoenix-project-book). I still recall a lot of what they talked about in that book day-to-day on the job. 

**As we're building a DevOps community, what is one thing about you that might surprise us?**

*Stephen*: I am an avid learner of rocket systems, especially SpaceX and NASA-related rockets and ships. I try never to miss a launch as it’s such a fascinating process. It kind of reminds me of application deployments!

**Great comparison. Yes, deploying software (without Octopus) is like launching a rocket, with manual checklists in place and a countdown, hoping that nothing blows up.**

**Stephen, it was a pleasure talking to you!**

*Stephen*: I enjoyed our conversation!


:::hint
If you’d like to feature in our series, [Inside DevOps](https://octopus.com/blog/tag/Inside%20DevOps), please reach out to [Joanna on LinkedIn](https://www.linkedin.com/in/joannawyganowska/) to set up time for a quick chat.
:::