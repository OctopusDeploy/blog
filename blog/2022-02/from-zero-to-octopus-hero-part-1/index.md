---
title: From zero to Octopus hero - Getting to know Octopus Deploy
description: Join Sarah Lean on her learning journey with Octopus Deploy.
author: sarah.lean@octopus.com
visibility: public
published: 2022-02-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 
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

Being able to handle software and infrastructure deployment, it makes it a great tool for Developers, DevOps Engineers and System Administrators. 

## Getting to know Octopus Deploy
Being a System Administrator at heart, I am always keen on understanding how a product runs.  With Octopus Deploy, there are a few ways of running it:

- Octopus Cloud, which is a cloud-hosted service that Octopus manages. 
- On-prem installation, which you install on a Windows server. 
- Container hosted installation using the Octopus Deploy Docker image on Linux. 
 
Over the past week, I have been working through how to [install Octopus Deploy](https://octopus.com/docs/installation) from scratch. 

It's been a great learning experience, getting hands-on with the technology. Building the infrastructure and my knowledge along the way. 

Octopus Deploy is straightforward to install on a single server configuration and set up a lab for learning purposes.  But, I wanted to learn more about how customers run it in a real-world scenario. 

I've spent a lot of time diving into running Octopus Deploy in a [highly available architecture](https://octopus.com/docs/administration/high-availability) in my environment. Looking at how to load balancers, database high availability, and shared storage configurations.  I had a few false starts, (mainly due to not thoroughly reading the documentation and assuming I knew what to do.  

 One of the benefits of working with virtual machines is you can delete them and start again when you make a mistake while learning a new product or concept. 

And I took advantage of that functionality!

My false starts taught me the importance of the Master Key to your Octopus Deploy environment and [how to recover](https://octopus.com/docs/administration/managing-infrastructure/lost-master-key) from not having a backup of it!

::hint
The Master Key is generated on installation and is used along with AES-128 to encrypt certain sensitive data in the Octopus database. Make sure you [back it up](https://octopus.com/docs/octopus-rest-api/octopus.server.exe-command-line/show-master-key) and keep it somewhere safe. 
::

It's always a great feeling when you get something working and see the results of your hard work.  I did fist pump the air when I finally got the proper configuration and could access my Octopus Deploy setup through the Load Balancer!

## Lessons Learned
Building and breaking my implementation has enforced the concepts of deploying Octopus Deploy for me. I'm building my knowledge from the ground up by working through exercises like this.  I'm not an expert in high availability configurations, but I feel secure in knowing the concepts and key points. 

Another area that tripped me up, was the shared storage.  This needs to be **configured and checked** before installing Octopus Deploy.  

You can install Octopus Deploy within the shared storage being in place.  But there will be some strange behavior if you do. So be sure to configure the [shared storage](https://octopus.com/docs/administration/high-availability/design/octopus-for-high-availability-on-premises#shared-storage) and check it is operational before you start the install.  

## Next steps
I'll be continuing my learning journey and have a plan in place for the next few weeks. I'll be learning the terminology that Octopus Deploy uses, setting up my first deployment, and learning best practices.   

Beyond that, I want to know how to look after my Octopus Deploy implementation. Build complex deployments, and learn how it integrates with other products. 

Be sure to check back to see how my learning journey is going, and let me know if you have any questions. 
