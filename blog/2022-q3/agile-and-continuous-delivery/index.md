---
title: Comparing continuous delivery and Agile
description: TBC
author: steve.fenton@octopus.com
visibility: private
published: 9999-99-99-9999
metaImage: placeholderimg.png
bannerImage: placeholderimg.png
bannerImageAlt: TBC
isFeatured: false
tags: 
  - DevOps
---

<!-- see https://github.com/OctopusDeploy/blog/blob/master/tags.txt for a comprehensive list of tags -->

Introductory paragraph that tells the reader why they should read on.

Compare the principles from continuous delivery with the principles from The Agile Manifesto.

CD was influenced by both lean and agile, and even took it's name from the first principle of The Agile Manifesto: "Our highest priority is to satisfy the customer through early and continuous delivery of valuable software"

People have tried to re-write the Agile principles a few times, but the original authors have left it alone.

Alistair Cockburn

 > ...it was a free-for-all, that is to say, there was no given purpose or agenda. Self-organization at it’s finest, held together by deep respect and generous listening on all sides. What emerged was what these exact 17 people could produce together at that particular moment in history. We agreed at the end never to update the manifesto for that exact reason.

https://web.archive.org/web/20170626102447/http://alistair.cockburn.us/How+I+saved+Agile+and+the+rest+of+the+world

When people rewrite it, we tend to get a well written, well thought through, single-person perspective. This is not representative of the community, though the best of these attempts have gained some momentum, such as Modern Agile, created by Industrial Logic's CEO, Joshua Kerievsky. Modern Agile places more emphasis on cultural factors.

https://modernagile.org/

https://heartofagile.com/

Agile describes something of a standard for a method to obtain - many different methods can be created and tested against the manifesto. For example, Extreme Programming (XP) and Dynamic System Development Method (DSDM) are both specific methods that have their own principles and techniques, but are both considered Agile because they pass the requirements of each of the manifesto principles.

Various methods contained different mixes of communication, process, and technical practices - though most were light on technical practices with XP being the notable difference.

Continuous delivery is a modern method that is focused on specific practices, with less emphasis on communication and process - however the practices have undergone intense scrutiny via research backed analysis, so we now know that they are worth the investment.

CD can be supplemented with a simple communication and process tool such as Kanban to co-ordinate the work, as it's highly complementary to many of the principles of CD and the development of a strong DevOps model for the relationships between practices (which places CD firmly in the picture) provides signposts to cultural and organizational capabilities that complement the practices of CD.

Let's look at the principles and then compare them...

The principles of The Agile Manifesto

https://agilemanifesto.org/principles.html

1. Our highest priority is to satisfy the customer through early and continuous delivery of valuable software.
2. Welcome changing requirements, even later in development. Agile processes harness change for the customer's competitive advantage.
3. Deliver working software frequently, from a couple of weeks to a couple of months, with a preference to the shorter timescale.
4. Business people and developers must work together daily throughout the project.
5. Build projects around motivated individuals. Give them the environment and support they need, and trust them to get the job done.
6. The most efficient and effective method of conveying information to and within a development team is face-to-face conversation.
7. Working software is the primary measure of progress.
8. Agile processes promote sustainable development. The sponsors, developers, and users should be able to maintain a constant pace indefinitely.
9. Continuous attention to technical excellence and good design enhances agility.
10. Simplicity--the art of maximizing the amount of work not done--is essential.
11. The best architectures, requirements, and designs emerge from self-organizing teams.
12. At regular intervals, the team reflects on how to become more effective, then tunes and adjusts its behavior accordingly.

CD principles

1. Build quality in
1. Work in small batches
1. Computers/Humans
1. Continuous Improvement
1. Everyone is responsible


Mapping

|                                                   | Build quality in | Work in small batches | Computers/Humans | Continuous Improvement | Everyone is responsible |
|---------------------------------------------------|------------------|-----------------------|------------------|------------------------|-------------------------|
| Early and continuous delivery / valuable software |                  | ✅                   | ✅                |                        |                         |
| Welcome / harness change                          |                  | ✅                   |                  |                        |                         |
| Deliver frequently                                | ✅                | ✅                     | ✅                |                        |                         |
| Work together                                     |                  |                       |                  |                        |                         |
| Trust and motivation                              |                  |                       | ✅                |                        | ✅                       |
| Face-to-face conversations                        |                  |                       |                  |                        |                         |
| Measure progress / working software               |                  | ✅                     |                  |                        |                         |
| Sustainable pace                                  |                  | ✅                     |                  |                        |                         |
| Technical excellence and design                   | ✅                |                       |                  | ✅                      |                         |
| Simplicity / scope reduction                      |                  | ✅                     |                  |                        |                         |
| Self-organizing teams                             |                  |                       |                  |                        | ✅                       |
| Reflect regularly                                 | ✅                |                       |                  | ✅                      |                         |

Most of the agile principles are covered by one of the CD principles


Technically speaking, CD principles don't explicitly cover the agile principles of working together and face-to-face conversations. The original CD book was written by Jez Humble and Dave Farley using different text editors, in different countries, with version control and automated builds. The authors and editors all worked remotely and asynchronously with a continuous delivery pipeline in place... i.e. they followed their own recommendations.

Rather than omitting the principles of working in the same space at the same time, CD specifically challenges that assumption. People should work together continually, but not necessarily in the same location. Remote working is seen as a competitive advantage, allowing modern employers to access talent that might not be available if geography was a key factor.

Conclusion is - it's aligned but not afraid to challenge some of the assumptions. Best not to be judgmental  of concepts that may have been more relevant in 2001 than they are two decades later.


## Body

The body of the post is where you share your hypothesis, how-to, or story.

### Sub headings

Use three ### to include H3 headings.

Use **Bold** text for UI labels, use single back-ticks for `parameters` and `filepaths`, and three back-ticks for code blocks:

```
Write-Host "Hello, World!"
```

Use the following (minus the back-ticks) to include images:

```
![Alt text, a description of the image](placeholderimg.png "width=500")*Optional caption text*
```
If including images, please include alt text. Alt text is primarily used to describe images to people unable to see them, and can be 125 characters max including spaces. You can also include an image caption if the reader would benefit from additional information or context.

## Conclusion

Close off the post by restating the main points of the post, share any closing thoughts, and invite feedback.

## Learn more

- [link](https://www.example.com/resource)

Happy deployments! 
