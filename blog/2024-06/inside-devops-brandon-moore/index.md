---
title: Inside DevOps with Brandon Moore
description: A series where we share lessons learned from those on the frontlines of DevOps. This post features  Brandon Moore, Senior Software Developer in the power industry.
author: tony.kelly@octopus.com
visibility: public
published: 2024-06-03-1400
metaImage: 
bannerImage: 
bannerImageAlt: Photo of Brandon Moore
isFeatured: false
tags: 
  - DevOps
  - Inside DevOps
---

This post is the next in our [Inside DevOps series](https://octopus.com/blog/tag/Inside%20DevOps), where we share lessons learned from those on the frontlines of DevOps.  

Hear from Brandon Moore, Senior Software Developer in the power industry, working for a large trade association. 

**What does DevOps mean to you?**

*Brandon:* To me, DevOps is a culture that an organization takes on to deliver faster, better, and more frequently to its customers. Who your customer is, or what you're delivering, depends on the role you're in at your organization. It might be end users, internal developers, or even business analysts.

To me, it encourages automation, experimentation, and challenging the norm in order to innovate and turn problems on their side.

The end result is an organization that has learned to function without silos and upped its delivery game.

**How did your DevOps journey start?**

*Brandon:* So that was a combination of things, really. In the Spring of 2018, I attended the Microsoft Build Conference in Seattle, and I attended a bunch of sessions involving build pipelines. I was working on a pretty important project at the time to standardize our build processes and pipelines and I think that’s when DevOps really started going mainstream.

The next project I took on was integrating an API test into the CI process, and that was a direct result of what I gathered from that conference and seeing the value of ‘shift left’ coming into play.

Obviously, there are faster iterations that come out of that, higher quality build products. And outside of work, I was working on a personal project and only had a limited amount of time to work on it. It got me looking into other areas of automation like Swagger API specifications, code generation, and containerization. It seemed to me like the universe was converging on the overarching theme of DevOps-centric solutions, and I just leaned into it.

**What are some DevOps best practices your organization has implemented?**

*Brandon:* For us, shift left has been a huge one. There's been a huge emphasis on saving time. And upping quality via pipeline automation. That's personally been my go-to for a lot of solutions and problems that I've been tasked to tackle, and we’ve vastly upped the quality of the deployments. We also embraced observability early on, getting an enterprise-class observability platform installed on our whole production-wide system. That was a game changer in terms of identifying certain issues and it really illuminated our blind spots.

**What was the observability platform you implemented?**

*Brandon:* Dynatrace. It helped us identify a struggling SAN product, our local SAN. It was being completely tapped out at nights in terms of I/O, so that allowed us to identify that bottleneck. 

It's enterprise-wide observability, so you have a developer (me) who was able to say, “Hey, this SAN, I/O channel is full,” when that wasn't technically a product I managed. But because we had that solution in place, it allowed that company-wide teamwork mentality to start.

**What’s the biggest challenge that Octopus has helped you with?**

*Brandon:* Octopus has been an amazing tool and there have been 2 main things that have helped us.

The first is getting our environmental sprawl under control. We're a larger team. We have a mix of AWS, serverless EC2, on-prem VMs, and database servers. Before Octopus, we had environment config files stored wherever the repo needed them. Octopus allowed us to untangle that and get everything lined up properly and centralized.

When Runbooks was introduced several years ago, that provided a way to automate tons of environmental processes, like backups and restores. And that's in addition to deployments. Before that, many things required coordination with other teams, like tickets, manual processes, and waiting. So it really broke down those barriers and let us own the end processes.

The second item was integrating with the ticketing system and the build system to provide automated feedback to Octopus that enabled better visibility into what's going into each deployment. So, we avoid having to go to other systems to understand the who, what, and when of deployments.

At the end of the day, all that centralizing rewards us with a single pane of glass – a living dashboard for all of our application deployments, regardless of what we’ve deployed where. As a result, our team’s quality of life is much higher and we're able to move quicker than ever before.

**I love runbooks! We use them internally in my team as well, and you very quickly get to a point where you wonder how you lived without them. Can you talk a little bit more about your experience with Octopus Runbooks?**

*Brandon:* I'm the heaviest user of Octopus at my company. To the point I want to put everything in there because I understand it now.

I support one product that has been around for over a decade. It's a commercial off-the-shelf (COTS) product, meaning we get the application binaries from a vendor, and it's always been considered impossible to automate. This was one of the biggest challenges I had.

I thought, “You know what? We can automate this, and we're going to use Octopus.” 
The suggestion kicked off an internal debate – is this right, is this feasible, is Octopus the correct solution?

But we did it, and now we have push-button deployments and supporting runbooks for something that was the most painful thing in the world for 15 years. It’s been amazing.

**What’s the most challenging part of DevOps?**

*Brandon:* Without a doubt, I’d say it's the culture shift. You need an organization that dives in headfirst, with leadership that fully embraces the mindset. 

Early in my career, a mentor taught me that it's natural to fear change. It's natural to fear things you don't immediately understand, and many people are also afraid of failure. In DevOps, once it's understood that this failure and the bumps in the road are part of the process, and you've created that innovation ‘safe space’, the real transformation can happen. 

The story I just shared with you about automating what was deemed the impossible, is a result of breaking down that culture barrier.

**What’s the most rewarding part of DevOps?**

*Brandon:* Transforming teams by changing how they work and witnessing the joy on their faces when they realize their job has fundamentally changed or it's easier. Hopefully, it gets easier with DevOps. That's what we all want.

A good example is automating the COTS product. We scored a huge win, and the team functions differently now. There’s a different vibe. It's a weight off people’s shoulders for a lot of things. And people can participate more in the industry this way.

I remember the look on my colleague’s face when he opened the Octopus build log and said, “Well, I'll be. It’s all there!”

**What advice would you give to someone starting their DevOps journey?**

*Brandon:* There are hundreds if not thousands of tools out there. You need to start by understanding their uses and the classifications.

I'm big on tool sets. You have to question how they support DevOps best practices. Are they observability, are they CI/CD, are they for change management? 

Don't be afraid to experiment with home labs or reach out to vendors with questions. People buy products and don't realize they can ask questions if they need to. 

After you get familiar with the tools at your disposal, you'll start identifying problems they can solve.

Don't be afraid to pitch new things. To start, focus on the tangible problems you can solve. Then you'll start seeing an impact in the areas that aren't as easy to measure, like culture.

**What DevOps trend have you been following lately?**

*Brandon:* So, GitOps was big!

I was disappointed to see what happened with Weaveworks. I attended a few of their presentations, and they were doing some pretty innovative things.

But I think overall as an industry trend, GitOps is here to stay.

I think it’s huge that Octopus picked up Codefresh. And I was just reading an article in The New Stack about how you have on-prem, GitOps, and Kubernetes all under one hood. I think that's going to be huge for centralization.  

But like I said, there are thousands of tools out there and I think we're starting to see who's here to stay. We’re seeing who has the right tools to get the jobs done. So there’s more maturity in tool selection.

**What DevOps book do you recommend?**

*Brandon:* The one that always stands out is [Accelerate by Nicole Forsgren, Jez Humble, and Gene Kim](https://octopus.com/devops/reading-list/#accelerate). It did a great job of setting expectations with examples, and defining the language concepts of DevOps.

It gives a pretty good definition of culture. It's like an intro to DevOps. That was the first book I read. Its principles of technical management and cultural practices are a good starting playbook for organizations. I particularly appreciate how they discuss empowering teams with tool sets and the resulting transformations.

**What's one thing about you that might surprise us?**

*Brandon:* The personal project I mentioned early on was actually a proof of concept for an IoT device in the dental industry. Early last year, I was awarded the patent related to that. So in addition to programming and DevOps, I can now add ‘inventor’ to my resume.

:::hint
If you’d like to be featured in our series, [Inside DevOps](https://octopus.com/blog/tag/Inside%20DevOps), please reach out to [Joanna on LinkedIn](https://www.linkedin.com/in/joannawyganowska/) to set up a time for a quick chat.
:::