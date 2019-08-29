---
title: Implementing DevOps in State Government
description: The story of how I implemented DevOps practices at a State government agency
author: shawn.sesna@octopus.com
visibility: public
bannerImage: blogimage-devopgovernment.png
metaImage: blogimage-devopgovernment.png
published: 2019-09-04
tags:
 - DevOps
 - Government
---

![Illustration showing an inifite feedback loop surrounding a government building](blogimage-devopgovernment.png)

## Introduction

We’ve all heard the phrase “the speed of government” when describing things that move slowly. Implementing change in government is possible, but it's all about slow and steady progress. 

Start-ups and government agencies both suffer from issues with standardization of builds, testing, consistent infrastructure, and reliable deployment processes. The biggest challenge with implementing a concept like DevOps is change. Let’s be honest, change is hard, but it doesn’t have to be bad. 

In 2011, I was hired as a Configuration Manager at a small US state government agency. I was tasked with automating the manual processes to improve the reliability of software deployment, reduce the length of time it took to deliver, and to try eliminating the need to deploy on weekends. This was a challenging mandate and it took years to achieve.

In this post, I'll cover the approaches I took to achieve this and cover some of the common pitfalls that you may face in a similar environment. 

- Prioritising and tackling the biggest problems first
- Building vs buying tools
- Communication and collaboration
- Constant progress and next steps
- The (eventual) result

## Prioritising and tackling the biggest problems first

The agency I joined had a lot of problems and the first thing I needed to do was learn how everything was currently operating and structured. Once I settled in and learned their processes, I was able to priotise my first steps to improve things. This agency had a number of in-house appliations and their deployment process left them wide open to problems. Web app builds were done on developer machines, zipped, and copied to a file share. Database changes were handled similarly by zipping up a bunch of scripts with a document explaining in which order to run them, then copying them to a file share for the DBAs to pick up. Inevitably, deployments would fail for any one of the following reasons:

- A developer neglected to mention there was a third-party dependency that needed to be installed on the web server. 
- The scripts for the database changes weren’t tested to make sure they worked with the current state of the production database.
- Good old-fashion human error. 

Things needed to change, badly.

I felt the highest priority was to start at the beginning of the process with the builds so I could eliminate the adage, “worked on my machine, ops problem now.” The team was using Microsoft Team Foundation Server for source control, so the build controller technology was already present. I installed the controller as well as a couple of agents so that all software was built against an independent machine. This quickly brought to light any dependencies that were present on the developer machines that were not available on the servers and needed to be installed. Though this helped identify what had to be installed on the server, the web admins still occasionally forgot to install the dependencies on all the servers and deployment failures still occurred. That issue didn’t get resolved for several years.

Next, I tackled web code deployments. I learned about Microsoft Web Deploy and wrote a small console application that used it to consistently deploy the web code. This required a small change in the build process to produce the zip file in a way that Web Deploy expected. Using web.config transforms, we got most of the way to automating web code deployments, but we still had to update connection strings manually. Not great, but it was a start.

For database code, I wrote another small console application that read an XML file that specified the order to run the scripts. Unlike the manual way of running the scripts, the console application ran the scripts within a single transaction and rolled back in the event of a failure. Not only did this method reduce the error rate, but it also reduced the time it took for deployments since the DBAs no longer had to open the scripts one at a time and execute.

With these two console applications, we reduced the failed deployment rate as well as reduced the time it took to complete them. The skepticism and “that’s how we’ve always done it,” mentality morphed into acceptance. Weekend deployments weren’t completely eliminated, but they were significantly reduced, which made for happier devs and operations folks.

The console applications were eventually combined into an automated deployment solution that consisted of a server, agents on target machines, and a web-based interface. Developers had the ability to pull directly from source control, build, and schedule deployments for a later date and time. This solution further reduced the failed deployment rate and sped up the deployment process all but eliminating the need for weekend work. Eventually, the developers outgrew the solution and needed something more than my programming skills could deliver.

## Building vs buying tools

The team was happily using my in-house deployment solution when a contractor gave us a demonstration of a tool he’d come across for automating deployments, Octopus Deploy. After the demonstration, one developer was adamant we needed Octopus. As the proud papa of my solution, I was reluctant to abandon my creation.  By this time, I’d taken on the additional responsibility of being the Data Team supervisor, so I had less time to code, and I could no longer keep up with feature requests from the development teams. Octopus Deploy had a team of developers doing this as their full-time job, and I was just one person with limited availability. The final nail in my solution’s coffin was when a developer showed me he had to answer 14 questions to set up a deployment.  Compared to Octopus, this was both cumbersome and inefficient. Setting my pride aside, I duplicated the functionality of my solution in only a few weeks with Octopus, and full Octopus adoption followed a few months later.

This illustrates the classic problem of building in-house tools vs buying commercial ones. I'm happy we chose an off-the-shelf solution in this instance. Sometimes it's an advantage to build and maintain in-house tools but it's important to ask the question and weigh up the costs. 

## From confrontation to collaboration

Development tools can be a great enabler of teams to automate processes and solve problems but people and communication are critical to the success of DevOps and achieving change. As we automated more processes, tension between teams began to ease as constant fire-fighting fell by the wayside. An organic byproduct was cross-team communication increased and became collaborative discussions versus heated finger-pointing.  Where once DBAs would respond with, “It’s not the database server; it has to be your code.” They started saying, “Let’s take a look at the execution plan and see if there can be some efficiencies gained.” Your problem became our problem.  Developers stopped coming directly to me, red in the face, yelling about how my team was impossible to work with.  Instead, they consulted the DBA team to see if a design might be improved, a stored procedure could be enhanced, or if they could come up with index recommendations.

This is a great example of development teams and operations teams working together. It was difficult at first but we slowly built trust and teams started worked togther more and more. This was one of the most rewarding parts of the journey. 

One of the greatest lessons here was to talk to separate teams first to get them on board and then bring them together to agree on new processes. No one likes surprises and this approach worked very well. 

## Constant progress and next steps

At this point, we had made great process and we had automated the vast majority of our development and deployment processes. The next step was to review our priorities across development and operations.  One issue that continued to plague us was inconsistent environments. I learned about Infrastructure as Code and was immediately on board with the concept. Having no experience in any of the existing technologies (Chef, Puppet, Ansible, PowerShell DSC, etc…), I decided to try PowerShell DSC (Desired State Configuration). I quickly learned why all of the PowerShell courses say something like, “... and then there’s PowerShell DSC, but that’s a whole course in itself.” 

Octopus Deploy gave me some great PowerShell experience, but DSC was definitely a different animal. After a bit of learning, I could demonstrate how to configure a bare metal server (a VM, to be honest) to functional IIS server in minutes. Not only that, I showed how I could combine the deployment power of Octopus Deploy with PowerShell DSC and push out configuration to servers just like an application deployment! Now that I had the web administrators on board, I turned to the database administrators. Working with the DBA team, we created a DSC script that would install, configure, and maintain SQL Servers and hooked that up to Octopus as well. The DBA team could now keep tabs on their servers and change things whenever they wanted. This reduced the friction between the Operations team and the DBA team.

DSC also reduced friction between operations and application development.  Operations no longer needed to be involved in the installation or configuration of either IIS or SQL Server; their job was solely focused on hardware and Virtual Machine (VM) health.  DSC was being executed through Octopus Deploy using service accounts, which meant non-operations personnel no longer needed administrator rights to servers, which made security personnel very happy.  Another aspect of DSC was the fact it was self-documenting and version controlled.  If operations needed to see what or how something was being configured, they could consult what was in version control.

## The (eventual) result

None of this happened overnight. At this point, we’re at the beginning of 2019 and close to the end of my career in state government, but I’d implemented Continuous Integration (CI) with builds that automatically ran whenever a check-in was performed. Most projects had implemented Continuous Delivery (CD), so after a CI build completed, it would automatically deploy to the lower-level environments for the testers and business analysts to begin their processes for approval. I’d automated the following;

1. Deployment of ASP.NET web code.
2. Windows Services.
3. Database deployments.
4. SQL Server Reporting Services (SSRS) reports.
5. SQL Server Integration Services (SSIS) packages.
6. Console applications.
7. Java applications to WildFly. 

I remember a compliment from a developer who said he loved the fact he could click merge, go get coffee, and when he got back, his app was deployed. 

## Conclusion

Introducing change and introducing DevOps concepts in a government organisation can be slow and challenging but it's definiely possible. I was able to be successful by prioritising and tackling the biggest problems first, buying tools to simplify and standardise, focus on communication and collaboration to get other teams on board and constant progress allowed us to tackle other priorities.
