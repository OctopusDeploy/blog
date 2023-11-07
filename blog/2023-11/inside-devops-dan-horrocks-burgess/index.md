---
title: Inside DevOps with Dan Horrocks-Burgess from DDA Software
description: A series where we share lessons learned from those on the frontlines of DevOps. This post features Dan Horrocks-Burgess of DDA Software.
author: steve.fenton@octopus.com
visibility: public
published: 2023-11-13-1400
metaImage: blogimage-insidedevops-danhorrocksburgess-2023.png
bannerImage: blogimage-insidedevops-danhorrocksburgess-2023.png
bannerImageAlt: Photo of Dan Horrocks-Burgess
isFeatured: false
tags: 
  - DevOps
  - Inside DevOps
---

This post is the next in our [Inside DevOps series](https://octopus.com/blog/tag/Inside%20DevOps), where we share lessons learned from those on the frontlines of DevOps.  

Hear from [Dan Horrocks-Burgess](https://www.danhb.co.uk/), co-founder of DDA Software, a full-service technology consultancy specializing in software development, DevOps, digital transformation, and cloud.

**How did your DevOps journey start?**

*Dan:* In 2013, I was part of a small to medium-sized team in the healthcare industry, striving to enhance our operational processes. The technology organization was structured with distinct units. We had a software development team, a dedicated testing team, a packaging team, and an operations team, who were also responsible for software deployment.

I wanted to improve the feedback loop as the silos meant it took a long time to show our work to the business. Instead of having to depend on external people to get the software to different environments, we wanted to be able to trigger deployments to pre-production environments and use the same process to deploy to production.

To solve this problem, I started writing PowerShell scripts to manage infrastructure, run tests, and deploy the software.

Although we achieved a success rate of approximately 60-70%, there was some resistance from certain teams in the organization. While some teams expressed a keen interest in adopting this approach, others weren't convinced. Some people were worried about how automation might impact their existing roles.

Unlike many stories you hear about DevOps adoption, the business people were really enthusiastic about the changes as they recognized the benefits very early.

**And what does DevOps mean to you?**

*Dan:* Initially, the focus of the DevOps movement was on fostering closer collaboration between development and operations teams. However, we soon recognized the importance of integrating other silos from business and security in the same way.

Over the past 5 years, the trajectory of DevOps has shifted, emphasizing culture, team dynamics, and interactions rather than being solely tool-centric. The human aspect of DevOps is crucial as these factors allow technical practices to impact business outcomes.

Companies who ask for help getting started with DevOps often want to treat it as they do when they engage an external DBA or security testing organization. They want someone to _build DevOps_ for them rather than make changes to their existing structures.

You can't shut the door on a DevOps office, and hope they'll solve this problem. DevOps requires broad changes to the process, technology, team and interactions, and culture. Everyone has a part to play in that.

**What are some DevOps best practices you see organizations implementing?**

*Dan:* Most teams have achieved a solid foundation with source control and automated builds. However, they often fall short in terms of comprehensive test automation and deployment automation. They fail to connect deployments back to the business need by joining deployments to work items.

Where deployments have been automated, they often fail the test of "repeatable, reliable deployments" because they don't bundle together the artifacts, process, and variables. This adds risk to the rollback process when they build a new artifact or deploy it using a changed process with different values.

Even where teams are able to use infrastructure automation, they're often excluded from observability. This prevents them from taking responsibility for the performance and cost-containment exercises necessary for healthy outcomes. Meanwhile, the operations team may be adding resources to try and solve problems that could be better fixed in software.

Most organizations lean towards extremes. Either a rigid and prescriptive heavyweight approach or total chaos. A better approach is to balance lightweight process with great professional discipline.

**What is the most challenging part of DevOps?**

*Dan:* In my experience, I've found that the hardest part of DevOps lies within the cultural shift required for the organization.

The benefits of DevOps, especially for developers, are often not immediately tangible and need a longer demonstration period. This lack of immediate gratification makes it difficult to persuade developers to embrace new practices when the advantages aren't readily apparent. Conversely, it's often easy to get support from the business side as you can demonstrate the improvements early.

Some attempts to embrace DevOps simply added an operations team member to the development team. While this is very literally making the two roles work together, it only really changes the prioritization of ops tickets, which isn't the best outcome.

Many people are hesitant to adopt DevOps because they've been burned by unsuccessful _Agile transformations_. Past efforts depended too much on a single person, such as a Scrum Master or Agile Consultant, to drive the changes. Without enthusiasm in the team, these efforts fall flat.

The context of software development is so critical to designing appropriate ways of working that the team is always best placed to work out what this looks like. They may have less knowledge of Lean, Agile, or DevOps - but they know far more about the problem at hand than anyone else.

**And what is the most rewarding part of DevOps?**

*Dan:* Seeing a team smoothly delivering software without all the stress and worry is a game-changer. It shows that tech and business folks can actually team up on complex projects without constantly freaking out. When there's trust and a good vibe, teams can do awesome work, and the business can nail those important decisions.

**What advice would you give folks just starting their DevOps journey?**

*Dan:* Start small and look for little opportunities to deliver obvious value so you can build up trust in the team. They need to see the process working before they'll move past the distrust learned from poorly executed past transitions.

Build on that incrementally. Keep the team on side and look for ways to get the business on board. You want to make sure the changes are sustainable over the long term. It may be tempting to move too fast, but you need to start from where you are and apply continuous improvements at a pace where the team can learn new skills.

What works for one team won't work for another, so listen carefully for context signals.

When we made dramatic changes to a team in the healthcare industry, it was important to take the business stakeholders along with us. They needed to see that the process served their needs, just as it made the developer experience better.

The easiest part is the tools. It's easy to justify the spend on Octopus as it's easy to calculate the benefits. Your decision-makers will understand articles and white papers on the return on investment of tools like this. It's harder to explain the benefit of the less tangible capabilities of DevOps, like culture and lean product management.

**Can you recommend a good DevOps book?**

*Dan:* Kanban From The Inside by Mike Burrows gives practical insights into the application of Kanban and the reasons behind the practices. It's a useful read for DevOps people as it covers things that happen prior to that all-important commit. It also has useful information for your continuous improvement process.

**What's one thing about you that might surprise us?**

*Dan:* I'm a huge Disney fan and I'm visiting 4 of the 6 global resorts this year: Paris, California, Florida, and Tokyo. I'm hoping to add Shanghai and Hong Kong later on.

:::hint
If you’d like to be featured in our series, [Inside DevOps](https://octopus.com/blog/tag/Inside%20DevOps), please reach out to [Joanna on LinkedIn](https://www.linkedin.com/in/joannawyganowska/) to set up a time for a quick chat.
:::

Happy deployments!
