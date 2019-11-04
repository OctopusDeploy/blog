---
title: Why we bet on Kubernetes 
description: Why we chose Kubernetes to host Octopus Cloud 
author: michael.richardson@octopus.com
visibility: private
published: 2020-01-01
metaImage:
bannerImage:
tags:
 - Octopus
---

Before reading this post, if you haven't already, please read the [opening post in the Octopus Cloud v2 series](https://octopus.com/blog/octopus-cloud-1.0-reflections).  It sets the scene, and is a great read.  

To recap, we have: 
- A cloud SaaS product for which there is strong demand 
- A pricing model where the revenue per customer doesn't come close to covering the AWS hosting costs  
- A non-optimized architecture which allocates a dedicated VM for each customer

Octopus had always been designed to be hosted on user's own hardware, not as a multi-tenant co-hosted solution. So when architecting Octopus Cloud there were many different paths available.  In accordance with the finest of engineering traditions, we started with the _Simplest Thing That Could Possibly Work_, which in this case was hosting each customer on a dedicated virtual machine.  This approach was a resounding success.  It removed many unknowns, leaving us with a clear problem to solve...

***Reduce hosting costs.***

What does a v1 customer cost? 

![Octopus Cloud 1.0 architecture diagram](../../2019-10/octopus-cloud-1.0-reflections/octopus-cloud-v1-architecture-diagram.png "width=600")

The major components of the hosting costs for each v1 customer are (in approximate US dollars):

- Database: **$9** 
- File Storage: **$23** 
- Compute (VM): **$50** 

This gives a total hosting cost of roughly **$82** US dollars per customer. 

The goal was to bring this below **$10**/customer.  

It was time to let the engineers off their leash. 

## The Options

We will talk more about the database and file-storage components in future posts.  Today we're going to focus on the compute costs. 

Every option was on the table, but in hindsight they can be separated into a few broad categories.  

It's worth mentioning that at this point in time Octopus is implemented as a full-framework .NET application which requires Windows. The HTTP server runs as a self-hosted [NancyFX](http://nancyfx.org/) app.

Also worth highlighting is a goal we had from the beginning: _Do Not Fork Octopus_. We very much wanted to maintain a single code-base for our self-hosted and cloud products. 

Wait!  One last disclaimer. The following were our opinions at the time. We are certainly not presenting them as facts. They were based on our unique set of circumstances and constraints. You may disagree with them.  They are probably incorrect (or at best only partially correct). And they would certainly have been different if made at a different moment in time. Consider yourself disclaimered. 

### Option 1: Single Multi-tenant Server 

We posed the hypothetical question:

_If we were building Octopus from scratch today, as a cloud-hosted SaaS product, what would it look like?_

And the answer was that we would almost certainly build it as a natively multi-tenant solution; in other words, a single web application which could serve all customers.  

This approach would have the major advantage of making hosting for the server easy-peasy! It would just be a regular web application. 

Unfortunately Octopus is more than just a web server: It is also a task runner. 

We could fairly easily imagine modifying the Octopus HTTP server to be multi-tenant. We could still have a database-per-customer, and determine the connection string to use for each request.    

The task-runner component would be trickier. Certainly not impossible, but there would be some significant renovations required.  

Polling Tentacles would also be a complication. We won't delve into the details of this here, but for the purposes of this post its enough to know that we would also have to demolish a few walls (and add a bathroom) to support polling tentacles in this architecture. 

The worrying thing is these renovations would be occuring while we were also trying to continue shipping regular updates to the product. This would leave us with the diabolical choice between 

- Keeping the hosted architecture refactor on a separate, long-lived branch.  Because who doesn't love merging long-lived branches with major architecture changes?! 

- Merging regularly, and risk destabilizing our self-hosted customers (you know, the ones that are actually profitable) 

If there was no alternative, we may well have gone this route.  And one day we still might. But we felt we could have our cake and eat it.  We believed we could drastically reduce our hosting costs, with minimal risk to the Octopus Deploy product. 


### Option 2: Windows Process Per Customer 

The gist of this approach is that we would run each customer as a dedicated process on a Windows VM.  The process would host the Octopus Server API and task-runner components.  Each customer would still have their own database.   

The big advantage of this approach was that very few changes were required to the Octopus product. It also gave us a lot flexibility with where we hosted (AWS, Azure, self-hosted, etc); VM's are a generic commodity.  

The big disadvantage of this approach is that we would have to orchestrate these processes ourselves.  Examples of the questions we would need to answer:

_When a new customer arrives how do we decided which VM to execute their process on?_   

Would we just use a static number, for example 10 instances per VM?  Would we consider CPU and memory usage?   

_How do we deal with the noisy neighbour problem?_

If a customer is using more resources than expected and is impacting other users on the same machine, would we relocate them? If so, to where? 

_When customers leave do we infill those slots?_

Many people use cloud Octopus for trials, which are later abandoned. This would result in sparsely populated VM's if we didn't reallocate, which over time would reduce the benefit of the project. 

Of course all these questions could be answered, but they would require writing and maintaining orchestration code that was not our core business. And this orchestration code would likely _not_ be cloud-agnostic, and would tie us to a particular vendor. 


### Option 3: Azure App Service 

What's the optimal number of servers to wrangle?  Zero!  

We could host each customer as an Azure Web App.   

The _huge_ advantage of this approach was no infrastructure to manage.

The disadvantage of this approach was similar to above: we would still have to orchestrate allocating users between Service Plans.    
Significant work would be required to re-architect Octopus to run as an App Service. Oh, and of course we were currently on AWS, not Azure (Spoiler alert for future posts: this was about to change). 
But these weren't even the biggest concerns.  Our biggest concern was leaving ourselves at the mercy of the Azure gods; the dreaded vendor lock-in.  What if Azure deprecated App Services? (admittedly unlikely) Or changed the pricing model significantly? (Perhaps not so unlikely)  

### Option 4: Kubernetes

Being in the deployment tool business, we watch new technologies in this space with interest.  Over the past years, we had watched Kubernetes go from being an interesting project that few people outside of the devops world had heard of, to one of our most popular feature requests.  So in the background of making this decision on our own hosting platform, a small team within Octopus were busy implementing [support for Kubernetes in the product](https://octopus.com/blog/kubernetes-containers-update). Building integration into the product of course involved much exploring, evaluating, and just generally playing with Kubernetes.  And so a pleasant side-effect was that we had some experience internally with Kubernetes, and advocates for considering it for our own needs. 

But there was a significant roadblock: Octopus ran on Windows.  

Support for Windows nodes in K8s was available in beta form; which in reality meant it was undocumented and had possibly worked once in laboratory conditions somewhere.  An incredible feat of persistence resulted in the proof-of-concept (PoC) team getting Octopus running on Windows in a Kubernetes cluster.  It suffices to say we did not feel we were walking a well-worn path; more like hacking our way through a jungle, never quite sure which direction we were travelling.  And the Windows nodes proved to be unstable, regularly dying unexpectedly.   

We ruled out Kubernetes + Windows as a technology not mature enough to bet on.

More than that, we ruled out containers on Windows in general.  As part of evaluating K8s, we were also evaluating running Octopus containerized on Windows. 
Containers on Linux are very elegant.  Running a container on Linux results in a single process executing on the host.  Running a container in Windows, in contrast, results in many systems services also being run.   

These services each carry their own memory overhead, makes monitoring a little harder, and is just generally yukky.   

But the real deal-breaker was while performing these expermiments we invevitably ended up spelunking the internet, looking for resolutions to problems, other peoples experiences, etc:  There were tumbleweeds.  We just didn't get the sense that enough people were running production work-loads on Windows containers to have developed that critical mass of experience.  

Linux was a different story.  Containers on Linux were a well-travelled road by this point, with good documentation and tooling.  And we had evidence that many of our own customers were running production workloads on Kubernetes.  But this would require porting the Octopus Server to .NET Core, and running it on Linux.  

## The Decision

The final two contenders standing were:

- Windows Process Per Customer 
- Kubernetes (Linux and .NET Core)

Both required significant effort. A key difference was where that effort was directed: 

- **Windows Process Per Customer:** Building the orchestration infrastructure to allocate and monitor Octopus server processes 
- **Kubernetes (Linux and .NET Core):** Porting Octopus to .NET Core and ensuring it can run in a Linux-based container

We didn't spend a lot of time trying to compare the effort involved in the two approaches.  Firstly because, well, we suck at estimating effort just as much as everyone else.  But more importantly, over time the difference in implementation cost would be amortized away. What were we left with? 

_And the winner is [drum roll]..._

***Octopus would be built against .NET Core, run on Linux, be containerized, and orchestrated by Kubernetes.***

We decided the effort to port Octopus to .NET Core was effort that we were going to spend at some point regardless (in fact the port had already begun). 

Kubernetes was built for the problem we were trying to solve. We've always preached using Octopus rather than trying to roll your own deployment automation, and spend the time saved on making your core software better.  This was a chance to heed our own advice. 

It was also an exciting chance to again drink our own champagne: we could take advantage of the kubernetes support we had built into Octopus at a large scale. 

## Postscript 

This post was set in the past. These decisions took place over a year ago now.  
