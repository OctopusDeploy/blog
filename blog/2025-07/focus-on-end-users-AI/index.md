---
title: Focus on your end users when creating AI workloads
description: Why it is important to focus on helping end users above all else when creating AI workloads.
author: bob.walker@octopus.com
visibility: public
published: 2026-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 
isFeatured: false
tags: 
  - AI
---

Recently I attended a conference targeted at CIOs, CTOs, and VPs of Technology.  As expected, there was a lot of sessions on AI and how they can help companies be more efficient.  The example given was the well-known use of AI in the hiring process.  Use AI as gatekeepers to quickly weed out all the unqualified candidates.  "Your human resources people won't have to wade through so many CVs and phone screen!"  

That use case improves the efficiency of human resources or your people team.  But that efficiency comes at the cost of the end users, the people you are trying to hire.  _Everyone hates_ how AI is used in hiring processes today.  Phrases like dystopian and Orwellian are common.  In this article, I'll discuss why it is important to focus on both the beneficiary users and end users of your AI feature.

## Beneficiary User vs. End User

A beneficiary user is a person who gets a benefit from leveraging AI.  The end user is the person who will use an AI feature to accomplish a specific task.

Going back to hiring process, the beneficiary user is the person in charge of wading through CVs and performing the initial phone screen. The end user is the person submitting their CV.  The person in charge of going through CVs benefits from AI by off-loading the repetitive work of screening unqualified candidates.  Imagine a job posting for a senior .NET developer but 30% of CVs submitted are tailored towards project managers.  You might think I'm exaggerating, but you'd be surprised.  As a former hiring manager who had to wade through CVs, I was shocked how many people were "CV Bombing" - applying for as many positions as possible.

Looking at Octopus Deploy, the beneficiary of our AI Assistant is the platform engineer.  The end user is the developer who uses the assistant to accomplish a particular task.  For example, you can ask the Octopus AI Assistant why a deployment or runbook run failed.  The AI Assistant will look at the failure message, and using our knowledge base, our docs, and the web, will come up with a reason why the failure occurred and suggestions on how to fix it.  Assuming the suggestion is correct, the developer can quickly self-serve a solution and never involve the platform engineer.  The platform engineer benefits because they can focus on high-value tasks instead of helping debug a specific deployment failure.  If the platform engineer didn't know the answer, they'd be going through our docs or doing a Google search.  

Now that we understand the two kinds of users, let's examine what happens when a person is both the beneficiary and the end user.

## Learning the wrong lessons from the success of ChatGPT

ChatGPT and other similar tools are in a unique situation where a person is both the beneficiary and the end user.  

One of many benefits of ChatGPT is that it is an evolution of search engines.  Before ChatGPT, you did a Google (or DuckDuckGo) search and got back a list of results.  The search engine ranked the results for you.  They had complex algorithms to find the best results based on their internal ranking system.  A cottage industry of SEO (Search Engine Optimization) sprung up to get higher results.  ChatGPT changed that by providing you the answer curated from the content from many websites.  It uses linear math to find the most likely result.

For common questions, with many sources more or less agreeing on the same answer, the results between Google and ChatGPT are close.  ChatGPT is not infallible, one time it insisted that Omaha, Nebraska was 29 nautical miles from Chicago, Illinois.  Google can be more accurate, but that is a result of maturity.  They've had 25 years to improve and iterate their search results algorithm.  

For this particular use case, ChatGPT is popular because of the interface.  Everyone is familiar with the Google Search box.  The results are where they differ.  Instead of forcing the user to click through pages of results hoping for the best, it generates and formats an answer that is easy to read.  Google Searches are very transactional, do a search, get a result, move on with your day.  With ChatGPT, the sessions are interactive.  You can ask additional questions and ChatGPT remembers the entire conversation.   

I'm only focused on the question/answer aspect of ChatGPT.  I know it can do so much more, including generating content, images, composing songs, and more.   

Unfortunately, technology companies seem insistent on learning the wrong lessons when looking at the success of other companies.  They see "people like prompts and providing answers or content to them, let's do that for [insert use case here]!"    

## An awful user experience and its impact

That wrong lesson has its roots in computer graphic adventure games from the 1980s/early 1990s.

My first computer game was [Space Quest III](https://en.wikipedia.org/wiki/Space_Quest_III) from Sierra.  Like computer games of that era, I typed in commands to get the on-screen character to perform an action.  There was no help guide or tutorial.  I had to figure it out.  My brother and I spent _weeks_ trying to get out of the first area.  We had to find the magic set of commands to execute in a specific sequence in specific areas.

Last year, I started the multi-month process of changing banks from a regional to a national bank.  The national bank offered a high-yield savings account while the regional bank didn't.  There were a few use cases where I had to call the national bank.  They have followed the latest trend in phone support.  Dial the number, a human sounding voice asks you want you need help with.  Tell them what is needed the response is "I'm sorry, I didn't get that" or "I didn't understand."  I needed to know the magic phrase to get help.  There was no clear escape hatch to get to an operator.

Their online AI help agent was no better.  The AI help agent was trained on their public docs.  If the answer wasn't in the docs, it couldn't help me.  Often, it referred me to calling their support line.  It created an endless cycle of frustration.    

That experience was so bad that I went back to the regional bank.  They proudly promote you'll talk to a real person when calling for help.  I would rather lose thousands of dollars over the span of years than to deal with the national bank's awful AI-based help system.

## Augmenting the end user experience

The problem is just like humans, AI makes mistakes.  Unlike humans, AI cannot make complex decisions.  It doesn't do well when something is outside the expected parameters.  Today's AI has to still undergo many more evolutions to become similar to [Tony Stark's Jarvis](https://en.wikipedia.org/wiki/J.A.R.V.I.S.) in the MCU.  We are still a far way off.  In a [recent study](https://ml-site.cdn-apple.com/papers/the-illusion-of-thinking.pdf) by Apple Computers, many popular LRMs (Large Reasoning Models) models couldn't handle puzzles (Tower of Hanoi, Checker Jumping, Block World and River Crossing) once the number of pieces increased beyond simple examples.  Because of that, many AI features aren't empowered to help beyond simple use cases.  It results in a frustrating experience for the end user.  

When you work under the assumption that AI is imperfect like it's creator, the end result can augment the end-user experience.  ChatGPT and the Octopus AI Assistant ARE NOT intended to entirely replace the current end-user interfaces.  If a developer cannot solve a deployment failure using the suggestions from Octopus AI they can still escalate to their DevOps or Platform Engineers.  If I want to know about an answer from ChatGPT, I can go to the source and read the details.  They augmented what was already there.  

That is very different from using AI in the hiring process or using AI based help agents.  Unless you know someone at the hiring company or the magic phrase for AI Agent based help, there are no alternatives.  They are replacement end-user interfaces.  

When AI is the sole gatekeeper, the end-user experience suffers.  I believe that is one of the main reasons why [IBM found](https://www.ibm.com/thought-leadership/institute-business-value/en-us/c-suite-study/ceo) that only 25% of AI initiatives have delivered the expected ROI over the past few years.  

## Considerations for the end user experience

When designing the Octopus AI assistant we started with multiple questions focused on augmenting the end-user experience.  We didn't want to "sprinkle AI" into the product and claim we had an AI strategy.  

1. What problem is the AI feature attempting to solve for the end user?
2. What is the fallback when the AI feature encounters an unknown use case?
3. What is an acceptable level of accuracy for the AI feature?
4. In the event the response is wrong, what is the escalation process for the end user?
5. How will the functionality be discovered?

The answers for deployment failure functionality of the AI Assistant are:

1. Often failures are a result of a incorrect configuration, transient error, bug in the script, permissions, or some other common problem.  In many cases, it is outside the direct control of Octopus.  Surface the information to the user, to enable them to self-service the fix, and increase time to recovery.  
2. Provide a generic answer and encourage the user to contact Octopus Support or their internal experts.
3. Reasonable accuracy is expected.  Errors can be caused by a variety of conditions outside the control of Octopus Deploy.  Provide multiple suggestions using publicly available documentation.  If none of them work, encourage the user to escalate to a human.
4. In the event the response doesn't help, provide link to Octopus Support or to contact their internal experts.  In either case, they will escalate to a human.
5. When navigating to a failed deployment or runbook run, the Octopus AI Assistant will provide a suggestion the user can click on to get the answer.

The focus the entire time has been "how can we take what we have and make it better?"  Not, "how can we ensure that Platform or DevOps engineers are never bothered again."        

## Conclusion

When an AI feature has a beneficiary user and end user, focus on providing a fantastic experience for the end-user.  AI features must augment the end-user experience. But assume that at some point the AI will be incorrect (just like a person is incorrect), and offer a clear escalation path.  Despite the many advances in AI, experienced people can handle complex scenarios.  When the end-user isn't considered, any the only focus is "improving the bottom line" it ends up creating an inferior replacement for an existing experience.  End users will only put up with so much before they decide to change.  