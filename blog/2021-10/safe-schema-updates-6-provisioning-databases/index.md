---
title: Safe schema updates - Provisioning dev/test databases
description: The first step towards safe production releases... safe dev/test deployments.
author: alex.yates@dlmconsultants.com
visibility: public
published: 2021-10-18-1400
metaImage: blogimage-provisioningdevtestdatabase-2021.png
bannerImage: blogimage-provisioningdevtestdatabase-2021.png
bannerImageAlt: Four developers working on one database causing failure, next to 4 developers working on a database each, with  only 1 failing
isFeatured: false
tags:
 - DevOps
 - Database Deployments
 - Deployment Patterns
---

This blog post is part 6 of my safe schema updates series. Links to the other posts in this series are available below:

!include <safe-schema-updates-posts>

We’re more than half-way through this series and so far we’ve kept it pretty theoretical. We’ve imagined a better world, but we’ve not yet discussed the practical steps required to get there.

That changes now.

From now on, we’ll be explicitly discussing the process of taking the sort of hellscape I discussed in part 1, and iteratively refactoring it so it’s safer to make changes and improvements. 

For the record, this journey probably isn’t going to be quick or easy. We’ll be discussing technologies, processes and ideas that are likely to be new to a lot of your existing team. Learning is hard and it takes most people some time to accept, learn, and embrace new technologies and processes.

We aren’t going to get bogged down in tutorials, code snippets or other fine details, but there will be plenty of links to more detailed materials.

## We never have enough environments

First things first: provisioning dev and test environments. As we learned in part 2, ["failure free operations require experience with failure"](https://octopus.com/blog/safe-schema-updates-2-resilience-vs-robustness#appreciating-the-nature-of-failure-in-complex-systems). It’s unlikely your team will be able to build resilient production systems if they can’t easily access a realistic development space where they can safely practice failure.

By beginning with rapidly deployable and disposable development and test systems, we nurture the capability to better test and rehearse the risky refactors that will become necessary later. What’s more, with respect to delivery lead times: 


> The first place where the constraint almost invariably resides, especially for traditional IT organizations that have shared operations […] is environment creation. We can never get enough of them, and whenever we really need one, we still have to wait forty weeks.

*Gene Kim, [Beyond the Phoenix Project](https://octopus.com/blog/devops-reading-list#beyond-the-phoenix-project)*

Thinking back to what we learned in the prior posts:

- Personal, rapidly deployable environments are more resilient because they are disposable and they isolate failure. If a developer breaks one, the failure does not affect anyone else, and the environment can be terminated and respawned with ease. What’s more, if a developer’s personal dev instance is broken, they know they’ve discovered either a problem that already exists in production, or there’s an issue with their own code. This leads to less doubt and finger pointing.
- Self-service environments reduce “it worked on my machine/worked in dev” issues, since all environments are built from a standard image, which is as “production-like” as possible. 
- Personal dev environments encourage continuous integration since changes do not need to be batched together as they progress through the various shared dev/test environments. Changes are decoupled from each other. This results in smaller integrations, safer deployments, better quality, less bureaucracy, and faster lead times.

If we want to reap these benefits, it’s essential that environment provisioning is fast and painless. It’s critical to avoid any delays created by dependencies on operations teams or approvers. 

Our goal is to create a form of realistic, off-the-shelf, pre-approved environment that developers and testers can rapidly spin up and throw away as needed. It should feel like “git clone f5” and take roughly as many seconds and keystrokes.

This goal applies to every component of any system. Since the database is so often a shared dependency on which everything else is built, in this post we will focus on the rapid, self-service provisioning of databases. However, some of these ideas and technologies may also help folks to spin up additional dependent systems at the same time, allowing either humans or automated test runs to build whichever bits are necessary to complete the task at hand.

Using the techniques in this post will result in generally lower administration/management overheads and a safer product. However, cost-saving should not be the primary goal. In fact, the dev/test hosting bill (taken in isolation) may well increase somewhat. For those who fear skyrocketing infrastructure costs, consider that most hosting platforms allow for spending caps. The budget approval process should be concerned with where to set the cap, rather than micromanaging any specific compute minutes.

Let’s get started.

## Infrastructure as code

If using a cloud database such as Azure SQL Database or Amazon RDS, this step may not be necessary. However, for the purposes of this post, we’ll assume our monolithic backend database is a SQL Server database running on some virtual infrastructure, either in some private cloud or a hosting provider such as AWS, Azure, or GCP.

In this case, you need to version control some script that allows a developer to spin up a new virtual machine or container, either on their own dev workstation, a private cloud or a hosting provider.

Rather than attempting to teach the entire topic of infrastructure as code in a few paragraphs, for any readers who are unfamiliar with the concept, I recommend you read Bob’s excellent series, which starts here:

[Using Infrastructure as Code with Operations Runbooks](https://octopus.com/blog/runbooks-with-infrastructure-as-code)

Your objective is to provide each developer or tester with an easy means with which to spin up the necessary infrastructure on which to build a test database.

## SQL Server

Once you have your server, you need to install SQL Server. I recommend you start by reading another one of Bob’s posts (which I’m forever referring back to!):

[Automating SQL Server Developer installation](https://octopus.com/blog/automate-sql-server-install)

Bob explains how to version control your SQL Server configuration and automate the SQL Server installation. After you’ve read that, if you are running on Windows, I encourage you to look at [Chocolatey](https://chocolatey.org/). It’s basically Windows’ answer to Linux’s [apt-get](https://help.ubuntu.com/community/AptGet/Howto). It achieves the same as Bob’s automation scripts with [less code](https://community.chocolatey.org/packages/sql-server-2019):

> choco install sql-server-2019 --params="'/CONFIGURATIONFILE:c:\git\myrepo\iac\sql\ConfigurationFile.ini'"

Finally, it would be remiss not to talk about [Docker containers](https://www.docker.com/resources/what-container).

You can run SQL Server in a Linux container on either Windows or Linux. Docker allows folks to spin up new SQL instances much more quickly and efficiently than installing it on the OS. This allows developers to spin up and tear down instances much more frequently and liberally, without provisioning a new dev machine. Apart from speeding things up, this potentially simplifies the whole process by removing the need to spin up and tear down new servers every time a developer wants to rebuild their environment.

In my opinion, the best place to get started with SQL Server containers is [Andrew Pruski’s excellent blog series](https://dbafromthecold.com/2017/03/15/summary-of-my-container-series/), which takes you from zero to chaos engineering with Kubernetes (and… Space Invaders). 

<iframe width="560" height="315" src="https://www.youtube.com/embed/HCy3sjMRvlI?start=1642" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Databases/data

By now, we’ve automated our infrastructure and our SQL Server install, but we’ve not yet set up our databases or any test data. It’s no wonder that developers who lack realistic test data tend to write poorly performing queries. The first time their code is tested at scale is in production!

Unfortunately, using raw production data in dev/test is rarely possible/practical. Before we go any further, we need to address two problems that need solving: privacy and scale. We'll deal with these in turn.

### Data privacy: making useful but safe test data available

Most people would consider it an invasion of privacy if all the developers that worked for their bank, health care provider, or supermarket had access to their personal financial, health, or purchasing data. It’s also worth considering that hackers are more likely to attack dev databases than production databases. Phishing emails can be remarkably convincing these days. Once the bad folks have gained access to a developer machine, the dev databases are often much easier targets.

In response to these concerns, data privacy legislation is trending stricter almost everywhere. Quite apart from legal sanctions, both traditional and social media love to pile on whenever the next business suffers a data breach. No-one wants to be the next [Equifax](https://www.youtube.com/watch?v=LyIEd5QVkyc).

Ask yourself: “Imagine someone uploads your dev database to a popular hacker site… are you worried?” If so, it’s likely that either you know there is sensitive data where it shouldn’t be, or you don’t know whether the dev database contains sensitive data. Neither is acceptable under many of the latest data privacy laws.

This is true regardless of whether you maintain a single, shared dev environment, or whether you are using many disposable environments. However, if going down the disposable infrastructure route, it’s important to recognize we are likely to be creating many copies of the dev data, potentially exacerbating data guardianship headaches.

Regardless of whether we have one shared dev environment, or a hundred, the databases in those environments should not contain any sensitive data. There are various types of data you might want to replace your sensitive data with. To summarize in a tweet:

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">People often ask me how to manage data on dev databases.<br><br>Here is my answer on a Post-It. What do you think?<br><br>Also, would you find it valuable if I wrote a blog post on this topic or should I write about something else instead? <a href="https://t.co/Yj0esIwEZE">pic.twitter.com/Yj0esIwEZE</a></p>&mdash; Alex Yates (He/Him) (@_AlexYates_) <a href="https://twitter.com/_AlexYates_/status/991334539132796931?ref_src=twsrc%5Etfw">May 1, 2018</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

For the purposes of this post, I’m going to propose that the ideal data for most dev and test purposes is production data, but with any sensitive records either removed or replaced in some way. This way, procedures can be tested with a representative scale and distribution of data, better highlighting potential issues in disposable environments. This should result in more problems being caught in advance and fewer production issues.

To achieve this, first we need to conduct a data audit and create a data inventory/dictionary/map. Data needs to be classified based on the level of data sensitivity. (This is already a requirement of many data privacy laws including the GDPR. You cannot protect your sensitive data if you don’t know where it is!)

In SQL Server there are a couple of ways to do this. For example, you could enforce a policy that all columns use an [Extended Property](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-addextendedproperty-transact-sql?view=sql-server-ver15) to define the level of data sensitivity. These properties are relatively easy to put in source control [and query](http://workingwithdevs.com/query-return-sensitive-data-sql-server/). Further, [data classification has been a built-in capability](https://docs.microsoft.com/en-us/sql/relational-databases/security/sql-data-discovery-and-classification?view=sql-server-ver15&tabs=t-sql) of SQL Server since 2016.

Next, we need to create copies of the database with all the sensitive data scrubbed out. I propose using your environment creation scripts (above) to spin up a temporary “staging” instance somewhere behind your production firewall. Then, on a regular schedule, you can restore your most recent production backups to this staging area and run some script or tool to remove all the sensitive data. (Bonus: regularly testing your backups is an important practice anyway.)

The data masking process might be as simple as replacing all names with “John Doe”, or you might use a more sophisticated process to create realistic but fake data. For example, you might like to look at the [dbatools Invoke-dbaDbDataMasking cmdlet](https://docs.dbatools.io/#Invoke-DbaDbDataMasking) (open source, simple, free) or [Redgate Data Masker for SQL Server / Oracle](https://www.red-gate.com/products/dba/data-masker/) (3rd party, sophisticated, not free). Investing effort here to create more realistic test data will result in better quality dev and test work later on.

If you are nervous about sensitive production data slipping through, here are a few suggestions:

- Add a test to your deployment pipeline that ensures all columns have a data classification in source control.
- Add a test to your deployment pipeline that ensures all sensitive columns have a corresponding masking script or rule.
- Add a smoke-test after your masking process to scan for sensitive-looking data, such as social security, credit card, or phone numbers.
- Add a check to your deployment pipeline that any changes to data privacy classifications or masking scripts get flagged for review by a senior developer or (if you must) a database administrator, security team, data privacy officer or *[through gritted teeth](https://octopus.com/blog/change-advisory-boards-dont-work)* a Change Advisory Board. (Yo! “Separation of roles” enforcement brigade: I see you.) All I ask is that steps are taken to ensure these reviews are conducted swiftly, without creating long delays.

Following the masking scripts, developers/testers might like to provide their own scripts to run on the staging instance. For example, to add a set of known test cases to the database or to add a SQL login with admin rights for the dev/test group.

When all these scripts have completed, we can backup the new “dev-safe” databases on the staging server and copy the backups to some shared location in the dev domain. (Then we can drop the staging database instance. It’s served its purpose.)

Note: data masking scripts can take a long time to run and may require a lot of compute resource. This is especially true if working through large tables, row by row, and creating temporary tables all over the place to maintain complicated foreign key references. Hence, the whole thing needs to be run as a scheduled job, creating fresh “dev-safe” backups each night/week/sprint etc.

Now, by the time a developer starts work in the morning, the “dev-safe” backups should be ready for them to use. The scripts they already created to spin up their dev environments can now be extended to restore the latest masked production backups. Now their dev environments are complete with relatively recent versions of the masked production databases.

### Data scale: making the production data smaller, faster and cheaper

We probably still have a significant practical challenge. Most production databases are pretty big.

It’s fairly likely that the production database will be too big to reproduce in full on a dev environment. If the production database is measured in the terabytes, it’s unlikely you want to pay for lots of dev and test servers, each large enough to host the entire thing. In addition, large databases can take a long time to restore. We want the process to take seconds, not hours.

This being said, your developers and testers could really benefit from access to large-scale and relatively recent data. If they never test their code against large, representative datasets, how can they be expected to anticipate the sorts of unintuitive performance issues that can take down the production databases that underpin your entire production estate?

This is where database cloning can help.

Tools like [dbaclone](https://github.com/sqlcollaborative/dbaclone) (open source, free) and [Redgate SQL Clone](https://www.red-gate.com/products/dba/sql-clone/) (3rd party, not free) use the virtualization features already built into the Windows operating system to create cheap, editable virtual copies of large files. 

Within a SQL Server development setting, typically an operations team would start by creating a dev-safe “image” of a large database (up to 64TB) in a shared location. Later, the developers can create “clones” as required. These clones are effectively pointers back to the original “image”. When first created, each clone only requires a few megabytes (regardless of the size of the source image). Hence, we can create almost limitless clones of the source image, almost instantly, on cheap, commodity hardware. 

The clever bit is a “differencing disk” which captures any changes a developer makes on a clone. This feels like magic, since each clone becomes its own editable copy of the 64 TB source image, even if the clones are running on a much smaller drive.

The main gotcha is that the size of the “differencing disk” will grow as you modify the file. Hence, changes to small objects, like views, procedures, or updates to a single row of data, are unlikely to make a big difference. However, if you reindex a large table, you could quickly run out of disk space. Clones are an ideal tool to use on small, disposable infrastructure, where the clones are located physically close to the source images.

As a party trick, I used to run this cloning tech on a loop at the end of a demo in my conference sessions. Within a few minutes I had over a thousand, editable copies of [the full StackOverflow database](https://www.brentozar.com/archive/2015/10/how-to-download-the-stack-overflow-database-via-bittorrent/) running on my laptop. My local SQL Server instance thought I had almost a petabyte worth of SSD storage on my 13-inch HP Spectre!

If you’d like to learn more about how to use containers and clones together, you might like to start by watching a session I delivered last year with Sander Stad, the maintainer of the dbaclone project:

<iframe width="560" height="315" src="https://www.youtube.com/embed/masJxBmgfqo" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

And, if all this wasn’t cool enough, check out [Redgate Spawn](https://spawn.cc/). It’s a hosted version of SQL Clone. While it’s still in preview, and I’ve not yet had the chance to use it myself, I’m really excited about the potential! It has the potential to replace a significant portion of the steps in this post with a single command.

If you’d like to get started, [I wrote in more detail about database cloning last year](https://octopus.com/blog/self-service-database-provisioning-with-octopus-runbooks-and-redgate-sql-clone), and I included a much more detailed walkthrough.

## Schema deployment

By now, we’ve hopefully got a regular batch job producing dev-safe data images for each of our databases and a bunch of automation scripts allowing developers to spin up SQL instances with realistic data on-demand. However, those databases will probably be a bit out of date.

The database schemas are not based on the latest version from source control, they are based on production. (And production may not yet have the latest dev changes). What’s more, since the new dev images are created in advance, they might be a few days or weeks old.

Before we can use our new dev environment, we need to deploy our latest source code from our main branch in source control over the top of the most recent versions of the “dev-safe” databases. Apart from this being a necessary step to bring our dev environment up to date, I love this exercise because every time a developer builds a new environment, they are effectively testing the next production deployment. Hence, if there are any issues brewing, you are likely to spot them well in advance.

I’m not going to talk through the process of automating the database deployment here, but the following resources might help:

- [Database deployment automation approaches](https://octopus.com/blog/database-deployment-automation-approaches)
- [SQL Server deployment options for Octopus Deploy](https://octopus.com/blog/sql-server-deployment-options-for-octopus-deploy)
- [Comparison Review: Microsoft SSDT vs Redgate SQL Source Control](https://www.brentozar.com/archive/2018/12/comparison-review-microsoft-ssdt-vs-red-gate-sql-source-control/)

## Speeding it all up

Our provisioning process is now complete. It has two parts:

1. On a schedule, we have a process that creates new dev-safe databases for developers to use. 
2. Developers can spin up their own dev environments as and when they need them.

However, the “git, clone, f5” experience is still likely to be a bit frustrating for the developers.

When a developer wants to run their code, it’s relatively quick to clone the repo, and it’s possible for them to run their script (or perhaps use an [Octopus Runbook](https://octopus.com/docs/runbooks)) to build a development environment. However, that script is likely to take a while to complete.

As a developer, I do not want to wait more than a minute, and ideally not more than a few seconds, to start running my code. However, my environment provisioning script has to do all of the following:

1.	Build a new instance and boot it up. (This, at best, is likely to take a few minutes.)
1.	Install SQL Server, and whatever else is needed. (Probably another 5-10 minutes. Much less if using containers.)
1.	Restore my database backups or clone the databases. (If using large backups, could take a while.)
1.	Deploy the latest source code. (Depending on the size/complexity of the schema and the deployment tool/process, this could take a few minutes.)

It wouldn’t be surprising if that all took a good 15 to 30 minutes, and potentially much more for large databases. That’s a lot of thumb-twiddling and it’s likely to make the developer nervous about breaking their dev playground. Do they really want to risk such a long delay to respawn if they make a mistake? Iterating over a design or testing multiple implementation options could take a long time if each evolution takes hours.

We might be able to speed this up with VM snapshotting. Alternatively, we could create a queue of dev environments in advance. I like the queue option best since it saves all the VM wizardry and is potentially faster. The dev environment is ready to go before the developer even requests it - all they need is the connection string.

## Summary

According to Gene Kim, the first delivery bottleneck that most folks hit in their DevOps transformation is environment creation. Many of the issues we witnessed in [Database delivery hell](https://octopus.com/blog/safe-schema-updates-1-delivery-hell) were the result of shared and inconsistent development environments. We learned that failure is normal, so it’s important to create systems where failure is safe.

It’s my hope, that the sorts of technical practices outlined in this post will allow you, dear reader, to move as much development and testing off your big, shared development/test environments onto dedicated environments where changes can be tested in isolation and merged as soon as they are ready for deployment.

This improved testing capability will come in handy as we move on to the next posts about near-zero downtime deployment and the strangler pattern for breaking up monolithic systems into more loosely coupled architectures. It's a lot easier to commit to complicated and risky refactors if you can test and rehearse the changes in safe, disposable dev and test environments.

## Next time

In the next post we’ll turn our focus to deployment patterns.

Throughout this series I’ve been advocating for smaller, safer, more frequent deployments. However, if those deployments require periods of downtime, it’s unlikely that we’ll be allowed to deploy as often as we wish. There’s no point deploying 10 times a day if each deployment requires an hour of downtime. In my opinion, near-zero downtime deployments should not be seen as some lofty and unrealistic goal. They should be considered a prerequisite for practicing true continuous integration and for the delivery of resilient systems.

Links to the other posts in this series are available below:

!include <safe-schema-updates-posts>

## Watch the webinars 

Our first webinar discussed how loosely coupled architectures lead to maintainability, innovation, and safety. Part two discussed how to transition a mature system from one architecture to another. 

### Database DevOps: Imagining better systems

<iframe width="560" height="315" src="https://www.youtube.com/embed/oJAbUMZ6bQY" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### Database DevOps: Building better systems

<iframe width="560" height="315" src="https://www.youtube.com/embed/joogIAcqMYo" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Happy deployments!
