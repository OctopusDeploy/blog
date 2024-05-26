---
title: Behind the scenes of the Octopus Extension for GitHub Copilot
description: Learn about how we integrated Azure AI and Copilot when making the Octopus extension
author: matthew.casperson@octopus.com
visibility: private
published: 2024-05-25-1400
metaImage: 
bannerImage: 
bannerImageAlt:
isFeatured: false
tags:
- Product
- Integrations
- GitHub Copilot
---

Much attention has been focused on the generative aspect of GPT. It is easy to be impressed by the ability to create eye-catching images and videos or write sensible sounding text from a few simple prompts. It is also easy to understand how AI would augment the workflows of writing or drawing tools.

But how does AI provide any benefit to more traditional business processes? The answer to this is far less exciting than being able to generate a highly detailed drawing of a kitten in a space suit eating rainbows from little more than that very description, but it is these business-as-usual workflows where AI can have the most impact.

I had the privilege of exploring this very question in partnership with GitHub to develop the [Octopus Extension for GitHub Copilot](https://octopus.com/docs/administration/copilot). Copilot extensions were [announced by Satya Nadella at this year's Build conference](https://youtu.be/8OviTSFqucI?t=2334) with Octopus being 1 of 16 extensions for the initial launch:

![Build keynote address](build-keynote.png)

* bringing octopus to you
* move beyond chatbots
* realtime RAG - smart AI, dumb search
* rethinking testing
* hallucination
* permission to say "I don't know"
* safety
* interactivity