---
title: Octopus AI Experiment
description: We're testing the idea of allowing an Octopus space to be queried in plain English with an AI integration and would love some feedback.
author: matthew.casperson@octopus.com
visibility: private
published: 2099-03-25-1400
metaImage: 
bannerImage: 
bannerImageAlt: 
tags:
- Product
---

Large Language Models (LLMs) have generated a lot of attention for their ability to draw life-like images and videos, write fluent text, and even generate code. But they also have a remarkable ability to query structured data with plain English prompts allowing anyone to extract meaningful information from complex systems without having to learn SQL or write scripts to consume data from APIs.

Traditionally, if you wanted to extract information from Octopus that wasn't directly exposed via the web based UI, you had to query the Octopus REST API to read and link the desired information. This is certainly achievable, but requires a familiarity with scripting languages and an understanding of the Octopus domain model.

[Octopus AI](https://chromewebstore.google.com/detail/octopus-ai-experiment/lpeediihgpakkfdiliphohbglloghlmi) is a time limited experiment to help us understand if exposing the resources that make up an Octopus space to a LLM can be used to solve meaningful problems an extract useful information without resorting to custom scripting.

It is a plugin for Chrome that displays an overlay exposing a prompt for entering plain text queries and a field displaying the answer. If you have ever used a service like ChatGPT, the interface should look familiar.

The prompt is parsed to identify the various resources that were mentioned, such as a project, feed, account etc. The resources are queried via the API using the credentials of the currently logged-in user. The prompt and the supporting context are then sent to an LLM to answer the question.

The nice thing about LLMs is that you can ask them to interpret the supplied context and display the results in a number of useful ways. Here I have asked the LLM to summarize the vulnerabilities reported in the deployment logs as a table:

![Octopus AI screenshot](octopus-ai.png)

Here I have asked the LLM to extract any links from the deployment logs:

![Octopus AI Screenshot](extract-links.png)

Here I have asked the LLM to list which steps the project variables are used in:

![Octopus AI Screenshot](find-variables.png)

The results aren't always useful. Here I ask it if the project has any manual intervention steps. The results were nowhere close to being correct (the answer is none of the steps were manual interventions):

![Octopus AI Screenshot](manual-intervention.png)

But there is a good chance queries like this could be made to work by massaging the instructions and context provided to the LLM. And this is where you can help.

By using the plugin we can see the kinds of questions people would like to ask of their Octopus space. This will help us better understand how to optimize the data sent to a LLM to make the responses more useful and accurate.

If you would like to participate, all you need is a cloud hosted instance of Octopus (sorry, on-premises instances won't work), install the [extension](https://chromewebstore.google.com/detail/octopus-ai-experiment/lpeediihgpakkfdiliphohbglloghlmi), click the Octopus icon, and start typing.

We do collect information from the users of this extension. Sensitive variables (like secret variables, feed passwords, account passwords etc.) are not collected, we don't prompt for an API key, and personally identifiable information or anything that looks like a password is also filtered out. We will, however, collect the query, answer, and context (i.e. the configuration of the space being queried) to help us refine the tool.

The [Octopus AI](https://chromewebstore.google.com/detail/octopus-ai-experiment/lpeediihgpakkfdiliphohbglloghlmi) page has an FAQ providing some additional information about this project, and we'll keep the extension updated for the duration of the experiment, so you can automatically get the latest changes.