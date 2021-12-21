---
title: From zero to Octopus hero - Getting to know Octopus Deploy
description: Join Sarah Lean on her learning journey with Octopus Deploy.
author: sarah.lean@octopus.com
visibility: public
published: 2022-02-01-1400
metaImage: blogimage-fromzerotooctopusheropart1.png
bannerImage: blogimage-fromzerotooctopusheropart1.png
bannerImageAlt: Mario-style illustration with Sarah winning a fight with a creature over shark infested waters.
isFeatured: false
tags:
 - DevOps
 - Product
 - Getting Started
---

My name is [Sarah Lean](https://twitter.com/Techielass) and I recently joined Octopus Deploy as Senior Solutions Architect in the Community team. 

This blog post is part 1 of a series about my learning journey with Octopus. 

I'm starting from scratch with Octopus Deploy, but I intend to go from zero to Octopus hero over the coming weeks.

We'll add links to other posts in the series as they become available.

## What is Octopus Deploy?

Deploying software and infrastructure in your environment is something all software engineers need to do, whether we're running on-premises or cloud environments. Introducing automation into the process can help to drive efficiency and consistency. 

Octopus Deploy is a tool that can help with your infrastructure and software's deployment and release management. 

Being able to handle software and infrastructure deployment, Octopus is a great tool for developers, DevOps engineers, and system administrators. 

## Getting to know Octopus Deploy
A system administrator at heart, I'm always keen to learn how a product runs. With Octopus Deploy, there are a few ways of running it:

- Octopus Cloud, which is a cloud-hosted service that Octopus manages 
- On-premises installation, which you install on a Windows server 
- Container-hosted installation, using the Octopus Deploy Docker image on Linux
 
I've been working through [installing Octopus Deploy](https://octopus.com/docs/installation) from scratch. 

It's been a great learning experience, getting hands-on with the technology, building the infrastructure, and building my knowledge along the way. 

Octopus Deploy is straightforward to install on a single server configuration and set up a lab for learning purposes.  However, I wanted to learn how customers run it in a real-world scenario. 

I spent time running Octopus Deploy in a [highly available architecture](https://octopus.com/docs/administration/high-availability) in my environment. I looked at:

- How to load balancers
- Database high availability
- Shared storage configurations

I had a few false starts because I assumed I knew what to do and didn't read [the documentation](https://octopus.com/docs/administration/high-availability).

A benefit of working with virtual machines is you can delete them and start again when you make a mistake while learning a new product or concept. And I took advantage of that functionality!

My false starts taught me the importance of the Master Key to your Octopus Deploy environment and [how to recover](https://octopus.com/docs/administration/managing-infrastructure/lost-master-key) without a backup of your Key.

::hint
The Master Key is created on installation and is used along with AES-128 to encrypt certain sensitive data in the Octopus database. Make sure you [back it up](https://octopus.com/docs/octopus-rest-api/octopus.server.exe-command-line/show-master-key) and keep it somewhere safe. 
::

It's a great feeling when you get something working and see the results of your work. I fist-pumped the air when I set up the configuration and could access my Octopus Deploy instance through the load balancer!

## Lessons learned

Building and breaking my implementation helped enforce the concepts of deploying Octopus Deploy. I'm building my knowledge from the ground up by working through exercises like this.  I'm not an expert in high availability configurations, but I now understand the concepts and key points. 

Another area that tripped me up was shared storage. This needs to be **configured and checked** before installing Octopus Deploy.  

You can install Octopus Deploy in the shared storage being in place, but there will be some strange behavior if you do. Make sure you configure the [shared storage](https://octopus.com/docs/administration/high-availability/design/octopus-for-high-availability-on-premises#shared-storage) and check it's operational before you start the install.  

## Next steps

I'm continuing my learning journey and have a plan for the coming weeks. I'll be learning the terminology that Octopus Deploy uses, setting up my first deployment, and learning best practices.   

Beyond that, I want to know how to look after my Octopus Deploy implementation, how to build complex deployments, and learn how Octopus integrates with other products. 

Be sure to check back to see how my learning journey is going. We'll add links to other posts in the series as they become available.

Let me know if you have any questions by reaching out to [sarah.lean@octopus.com](mailto:sarah.lean@octopus.com) or leaving a comment below.

Happy deployments!