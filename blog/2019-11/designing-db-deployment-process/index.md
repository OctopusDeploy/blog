---
title: How To Design an Automated Database Deployment Process
description: This article will walk you through how to design your ideal automated database deployment process 
author: bob.walker@octopus.com
visibility: public
published: 2019-11-11
metaImage: 
bannerImage: 
tags:
 - Engineering
 - Database Deployments
---

Automating database deployments was the final piece of the CI/CD puzzle.  Once that happened, I went from 2-4 hour deployments to 10-minute deployments.  When I started Automating Database Deployments, I tried to fit tooling into the existing process.  That existing process was built over time to facilitate manual database deployments.  Essentially, how to get SQL files to DBAs to run them on Production.  I didn't realize the entire process had to change.  A new process had to be designed around facilitating automated database deployments.  

In the next couple of articles, I will walk you through how to design an automated database deployment process.  In this article, I am going to focus on core concepts.  If you want to skip ahead here are the links to the other articles:

- Designing Automated Database Deployments Process Use Case
- Implementing a Database Deployment Process with Octopus Deploy

!toc

## What to avoid
I've made many mistakes along the way.  Below is a list of hard lessons I learned automating database deployments.

- Don't include everyone at the start.  Create a small workgroup of 2-4 key people, such as DBAs or Database Architects.  Pick a pilot team to work through the kinks.  That team should send 1-2 people to the workgroup (bringing the total to 4-6 people).  The workgroup should be empowered to make decisions.
- Don't spend weeks or months trying to design the perfect process.  The workgroup should meet for a one to two-day kick meeting to create a draft of the ideal process.  After that, schedule regular check-ins to see how the pilot team is doing.
- Don't break your focus during the kick-off meeting.  Block out one to two days and don't allow open laptops, except to look something up to answer a specific question.  
- Don't focus on specific tooling.  Focus on the core concepts of the tooling available.  For example, modern version control software supports branching along with a review process for merging.  Every database deployment tool has a way to run it from the command line.  
- Don't try to automate rollbacks at the start.  You can spend the entire two days at the kick-off meeting trying to figure out all the possible scenarios.  You are no worse off if you wait.
- Don't be afraid to ask for help if you get stuck.  When I was working for a previous company, we asked Redgate for help, and they walked us through this process.  There are consulting firms, such as [DLM Consultants](http://dlmconsultants.com/), which can help. 
- Don't make decisions in a vacuum.  The workgroup should be transparent and solicit feedback at key points.  Provide regular updates to everyone.   

## Kick-Off Meeting Day 1
As stated earlier, the kick-off meeting should last one to two days.  The first day is going to be focused solely on the deployment process.   The existing process needs to change.  In some cases, a completely new process needs to be created.  

But you have to start somewhere.  Write down the existing process, so everyone is on the same page.  Include all the steps it takes for a change to make its way from developer to Production.  While writing, the process was written down, focus on answering these questions.

1. Who are the people involved in the process?
2. What permissions do they have?
3. Why are they involved?
4. Which environments have a different process?
5. Why are they different?
6. What happens when the script fails to run?
7. Why to scripts typically fail?
8. Who reviews the scripts and when?
9. Who needs to be involved with each deployment?
10. What isn't working, and what needs to change?

With those answers in hand, it is time to start working on a rough draft of the ideal process.  When designing the process, stay away from specific tooling terminology.  Say `all database changes will be reviewed by a database developer during the merge process` instead of `all database changes will be reviewed by a database developer via a pull request prior to merging into master`.  At first blush they look identical, but `pull request` and `master` are terms common to Git. 

Hopefully, by the end of the day, you have a working draft of the process.  This is a pre-alpha draft; it is okay if there are holes in it.  

## Kick-Off Meeting Day 2
The second day is about refining that process for the pilot team to implement.  Now that you have a rough idea of what you want to do, it is time to research the tooling out there.  Refine your process as you research the tooling.  It is okay to add steps, remove steps, or move them around as you learn more.  

When it comes to database deployment tooling, there are a lot of options.  For SQL Server, there is Redgate, DbUp, Apex, SQL Server Data Tools for Visual Studio (SSDT), RoundhousE, and Flyway, to name a few.  It is very easy to get analysis paralysis — especially when doing a side by side comparison of the tooling.  

Using the ideal process, identify two or three critical features, the tooling must-have.  Don't forget to think about who will be using the tooling in their day to day life.  The skill and comfort level could vary significantly.  For example, a DBA is more likely to be comfortable writing schema change SQL statements than a junior C# developer.  Here are some questions to help tease those requirements out.  Leave features every tool supports off of the list.  It is a given any tool that will be able to save to source control in some fashion — no need to include that.

1. [Model Based (Desired State)](https://octopus.com/blog/automated-database-deployments-iteration-zero#model-driven-approach) or [Change Driven (Migration Scripts)](https://octopus.com/blog/automated-database-deployments-iteration-zero#change-driven-approach)?
2. Should it integrate with existing tools such as SQL Server Management Studio (SSMS) or Visual Studio?
3. How are database changes detected and saved to source control?

In some cases, a tool meets two of the three critical requirements.  That tool is in consideration because it is free.  It is hard to argue with free.  There are some questions to consider when a free tool is considered that meets 2/3 of the requirements vs. a paid tool that meets 3/3 requirements.  

1. How much slower will adoption be because of that missing feature?  
2. Is there something that can be done to augment the free tool to get to 2.5/3 requirements met?  
3. Has the company attempted to use that free tool in the past?  Why wasn't it adopted?

I had a similar "free vs. paid" debate with Redgate vs. SQL Server Data Tools for Visual Studio (SSDT).  To purchase Redgate for 100+ developers, it would cost well into six figures.  Meanwhile, SSDT was free.  But SSDT integrates with Visual Studio, not SSMS.  Several teams attempted to adopt SSDT in the past, and all of them eventually abandoned the effort.  Too many people preferred to make their changes in SSMS, not Visual Studio.  They ended up with a weird multi-step process to get the changes into SSDT.  I'm not saying SSDT is a bad tool.  I'm saying that it didn't work for our specific needs.

## Pilot Team, Iterations, and Early Adopters
After the kick-off meeting and tooling research, it is time for the pilot team to take over.  Their goal is to implement the tooling and process, so it deploys through all environments to Production.  Along the way, they are providing valuable feedback to the workgroup and discusses what iterations need to be made.  The pilot team should not be shy about providing that feedback.  Any minor annoyance will be multiplied exponentially during general adoption.  

After a period of time, and depending on the company size, roll out the process to early adopter teams.  This helps refined the process more.  Assumptions or unintentional short-cuts made by the pilot team are found.  Additional scenarios are discovered and included in the process.

## General Adoption and Building Trust
The general adoption phase will be "moving a lot of cheese" for a lot of people.  Expect to get push back.  Databases are the key component of most applications.  A bad script can result in ruined days or weeks.

It is important to build trust in your process.  

I've found the two best ways to build trust in the process is manual verification and pilot teams/applications.  For example, generate the delta script and have a DBA review prior to deploying to QA.  A pilot application or pilot team is great at helping prove out the process.  They can work with the group who drafted the process to find the tooling to implement it and deploy it to Production.  Doing so finds the pain points in the proposed process and allows everyone to iterate on the process quickly.  

When it comes time for other teams to implement the process, they can include similar manual verification steps.  

Don't be surprised when you have to iterate on the process some more.  Every team and application is unique.  They might implement a database feature you haven't come across.  

## TL;DR;
To quickly summarize:

- Create a small team or workgroup to define the process.  Include representatives of each stage of deployment (developers, DBAs, etc.).  Should be no more than 4-6 people.  
- Identify the pilot team or application to include in the workgroup.
- Kick off the workgroup with a 1-2 day meeting
    - Write down the existing process, identify key people, pain points, and what needs to change
    - Draft up the ideal deployment process
    - Research tooling
    - By the end of kick-off, the pilot team should know what needs to be implemented and what tools to use
- The pilot team implements the new process
    - Deploy all the way to Production
    - Iterate on process
    - Once successful for a period of time, see if anyone would like to be an early adopter
    - Iterate on the process with early adopter teams
- General Adoption
    - Focus on building trust with the process
    - Roll out to multiple teams
    - Iterate when pain points are found

## Conclusion

This article covered a lot of high-level concepts.  In tomorrow's article, I will walk through a time when I followed a similar process.  It will hopefully ground this article with something tangible.  

Until next time, Happy Deployments!