---
title: Takeaways from Crowdstrike’s Outage
description: Lessons and takeaways we can learn from Crowdstrike's outage to apply to our development practices
author: bob.walker@octopus.com
visibility: public
published: 2024-12-31
metaImage: 
bannerImage: 
bannerImageAlt: 
isFeatured: false
tags:
  - DevOps  
---

Everyone is by now familiar with what happened on July 19th, 2024\.  A bug in a Crowdstrike configuration file, or "Template Instances," caused the Crowdstrike Falcon Sensor to crash.  Unfortunately, the Crowdstrike Falcon Sensor is configured as a Windows kernel boot driver.  Because that driver crashed, Windows did what it was supposed to do when a critical driver crashed: it stopped working and showed the dreaded Blue Screen of Death.

If you watched David Plummer’s video [Crowdstrike IT Outage Explained by a Windows Developer](https://www.youtube.com/watch?v=wAzEJxOo1ts) or his follow-up [Crowdstrike Update: Lessons Learned from a Microsoft Engineer](https://www.youtube.com/watch?v=ZHrayP-Y71Q) along with reading Crowdstrike’s [Preliminary Post Incident Review](https://www.crowdstrike.com/blog/falcon-content-update-preliminary-post-incident-report/) (PIR), there is a deeper philosophical discussion around third-party kernel drivers.  While perfectly valid, I’m neither expert nor qualified to discuss that topic.  And rather than adding to the chorus of "Crowdstrike should’ve done this" in this post I’m going to use those resources to provide context and takeaways we can apply to our teams and companies.

## Background Context

The Windows operating system has two modes: user and kernel. The kernel is responsible for performing low-level operating system tasks. For good reason, the access to the kernel is restricted.  Most software that needs access to the kernel (or the underlying hardware) will often run in user mode, make an API call to the kernel, wait for the kernel to act, and then return the appropriate information.  If you want to know more, please see David Plummer’s [Crowdstrike IT Outage Explained by a Windows Developer](https://www.youtube.com/watch?v=wAzEJxOo1ts) video and the [Architecture of Windows NT](https://en.wikipedia.org/wiki/Architecture\_of\_Windows\_NT) on Wikipedia.

Drivers control the various hardware and can run in user or kernel modes. Crowdstrike’s Falcon Sensor is a kernel driver. For obvious reasons, Microsoft doesn’t let any driver run directly in the kernel without some verification.  Microsoft runs the updated driver in their labs for testing and verification.  Before that, Crowdstrike runs updates through its extensive testing suite.  Assuming everything passes, the driver is signed and released to the public.  

The deployment process of that sensor is quite extensive, as detailed by Crowdstrike in their PIR document.

> This culminates in a staged sensor rollout process that starts with dogfooding internally at CrowdStrike, followed by early adopters. It is then made generally available to customers. Customers then have the option of selecting which parts of their fleet should install the latest sensor release (‘N’), or one version older (‘N-1’) or two versions older (‘N-2’) through Sensor Update Policies.

As David Plummer points out in his videos, getting a driver update through Microsoft’s certification process is not trivial and takes time. Changing a driver will alter the digital signature, as both Crowdstrike and Microsoft perform testing. 

With new threats appearing every day, an anti-virus company like Crowdstrike needs to react quickly. Crowdstrike has thought of that and can release an update to their definitions and configurations without changing drivers. This is known as "Rapid Response Content."

> Rapid Response Content is used to perform a variety of behavioral pattern-matching operations on the sensor using a highly optimized engine. Rapid Response Content is a representation of fields and values, with associated filtering. This Rapid Response Content is stored in a proprietary binary file that contains configuration data. It is not code or a kernel driver.

> Rapid Response Content is delivered as "Template Instances," which are instantiations of a given Template Type. Each Template Instance maps to specific behaviors for the sensor to observe, detect or prevent. Template Instances have a set of fields that can be configured to match the desired behavior.

> In other words, Template Types represent a sensor capability that enables new telemetry and detection, and their runtime behavior is configured dynamically by the Template Instance (i.e., Rapid Response Content).

> ***Rapid Response Content provides visibility and detections on the sensor without requiring sensor code changes (my emphasis).***

The Rapid Response Content is published to their Content Configuration System hosted in the cloud.  The Falcon sensor (installed locally) will download those updated files and write them to disk on the host.  

Any changes and additions to the Rapid Response Content are tested via "stress tested across many aspects, such as resource utilization, system performance impact and event volume."

## What Happened

A malformed Rapid Response Content configuration file made it to the Crowdstrikes Content Delivery System in the cloud.  Servers and workstations downloaded that malformed file from the cloud.  The sensor tried to load and run the malformed file, but it encountered an malformed file it couldn’t handle gracefully and crashed.  

Because it was a kernel driver, the entire operating system crashed.  David Plummer stated that this is a feature, not a bug.  It is a safety precaution, as allowing the OS to continue to run with a failed kernel driver could cause more damage. It’d be like continuing to run your car after all the oil leaked out.

The abbreviated timeline is:

* February 28th, 2024: Sensor 7.11 is released, introducing and adding a new mechanism to detect attacks against named pipes.  
* March 5th, 2024: A stress test using that new code was performed and was validated for use.  
* March 5th through April 24th, 2024: Four configuration files for the new functionality were released to production, performing as expected.  
* July 19th, 2024: Two additional configuration files were deployed. One of these files was malformed.  But both passed validation due to a bug in the content validator.  

Interestingly, there were tests to verify the validator (a test of the tests). They felt confident in their tests because they had successfully deployed other configuration files.

> Based on the testing performed before the initial deployment of the Template Type (on March 05, 2024), trust in the checks performed in the Content Validator, and previous successful IPC Template Instance deployments, these instances were deployed into production.

## Repeating Mistakes of the Past

Sadly, this is not an isolated incident. Earlier this year, Crowdstrike caused similar issues [with Linux](https://www.theregister.com/2024/07/21/crowdstrike\_linux\_crashes\_restoration\_tools/), specifically Debian-based distros. However, it was underreported because its impact on people wasn’t as prominent as the Windows outage.  Similar to the Windows Outage, testing is a primary cause of the issue.

> CrowdStrike acknowledged the issue a day later, but it took weeks to analyze the root cause. The analysis revealed that a specific Debian configuration was missing from their testing matrix.

Similar issues happened a month or so later with Rocky Linux.  

## Future Mitigation

In the takeaways section below, I list what we can learn from their outage.  To be fair to Crowdstrike, they have listed numerous mitigation strategies in their PIR.  I won’t harp on them that they should’ve performed this test or deployed a certain way.  Based on their PIR, they already know.  

* Improved testing: local developer testing, content update and rollback testing, stress testing, and more.  
* Additional validation checks are required for the content validator (more testing is needed for the testing tool).  
* Improve error handling in the content interpreter  
* Improved deployment: implement a canary style deployment, improved monitoring during that rollout, and provide customers greater control over the delivery of the Rapid Response Content  
* Third-Party Validation: independent third-party security code reviews and end-to-end processes.

## Takeaways

The outage on July 19th, 2024, was [a disaster](https://www.merriam-webster.com/dictionary/disaster).  Outages to 911 and global ground stops of major airlines are very serious.  Like many disasters, there isn’t one specific thing we can point to; it is a combination of issues.  And like many disasters, there wasn’t malicious intent.  No one at Crowdstrike woke up on July 19th and said, "I hope I can bring the earth to a standstill today\!"

I am not privy to Crowdstrike's internal workings, specifically its internal programming standards, SDLC, and CI/CD processes. As such, I don’t think it is fair for me to comment on what they should or shouldn’t change internally.  

However, we can learn from their mistakes and reflect on changes we can make in our teams and companies.

### Input validation for edge cases is critical

In Crowdstrike’s PIR, they mention needing to "Enhance existing error handling in the Content Interpreter."  Earlier in the PIR, they mention that "The Content Interpreter is designed to gracefully handle exceptions from potentially problematic content."  They have input validation in place but were missing edge cases.

It is great they had input validation.  However, there will always be edge cases.  [Injection](https://owasp.org/Top10/A03\_2021-Injection/) is \#3 on OWASP’s top 10 list for a reason.  Often, it is a failure of imagination or something that seems so preposterous that it doesn’t merit a test.  If you are writing input validation, have an attacker mindset.  Try to break your system.  Have others on your team do the same.

### Test with real-world data

Crowdstrike’s PIR provides some detail on how they test these configuration updates.  

> Newly released Template Types are stress tested across many aspects, such as resource utilization, system performance impact and event volume. For each Template Type, a specific Template Instance is used to stress test the Template Type by matching against any possible value of the associated data fields to identify adverse system interactions.

These configuration files help detect new attack vectors and viruses.  Interestingly, they don’t explicitly state they write an attack specifically for the sensor to detect on a Windows server or workstation.

Much time is spent writing unit and integration tests and configuring the external data to generate a repeatable result. For unit tests, we use fakes and mocks. Data is injected into a database or written to a file system for integration tests for consistency.  That consistency is needed to prevent the dreaded "flaky" test.  

However, that data is only as good as the imagination of the person who created it.  Nothing can replace real-world usage.  Remember, users are the ultimate bringers of chaos.

While working at DTN, I was tasked with rewriting a critical command processor. External sites would send commands following the PIDX protocol to our servers. The command processor read the database and determined whether to send a confirmation or denied message.  When I finished the new version, it had many unit and integration tests covering all the possible scenarios our QA team and I could think of.  However, the persistent fear was that it would fail when we switched from the legacy system to the new one.  So, I wrote a testing application called the "Chaos Monkey" that read the commands from the previous week, sent them through the legacy and new systems, and compared the responses.  Any mismatch in the responses would be logged for analysis.  The first time I turned it on, I found configurations I hadn’t dreamed of.  

Does something like this work for all applications? Absolutely not, but there are alternatives. When working at Farm Credit, we would refresh our staging environments with sanitized data from production. Our DBAs and Web Admins created an entire workflow to copy production to staging and remove the necessary PII, so we had real-world data to test with before production.  

Putting your application through testing scenarios using real-world data will help you find bugs before they reach production.  That may or may not be feasible for every application, but it is worth considering.

### Dogfood when feasible

Another interesting tidbit from Crowdstrike’s PIR is the difference between pushing a Sensor update and Rapid Response Content.

> The sensor release process begins with automated testing, both prior to and after merging into our code base. This includes unit testing, integration testing, performance testing and stress testing. This culminates in a staged sensor rollout process that starts with ***dogfooding internally*** (my emphasis) at CrowdStrike, followed by early adopters. It is then made generally available to customers. Customers then have the option of selecting which parts of their fleet should install the latest sensor release (‘N’), or one version older (‘N-1’) or two versions older (‘N-2’) through Sensor Update Policies.

> Template Instances are created and configured through the use of the Content Configuration System, which includes the Content Validator that performs validation checks on the content before it is published.

From that, it appears Crowdstrike dogfood's the sensor updates but not the Rapid Response Content update.  

Dogfooding is the act of using your application every day. Internally, at Octopus, we use Octopus Deploy to deploy Octopus Deploy. The instance we use (deploy) is updated daily (sometimes multiple times) using the latest build. We also have several employee instances that are updated before our customers. The core idea is to help find showstopping bugs before they reach the public.  

Whenever possible, you should dogfood your applications.  Not only will you find showstopping bugs, but you’ll also find annoyances, quirks, and other miscellaneous bugs that automated tests might miss.  

### Adopt Canary or Phased Rollouts for mission-critical applications

Once a Rapid Response Content file is made available in Crowdstrike’s Cloud, clients will download it.  

When talking to our director of security, he said his laptop suddenly got a BSOD around 2:30 PM Friday, Australia time.  He rebooted the machine, and it threw another BSOD.  Around that time, our cloud platform engineers noticed our Windows Dynamic Workers crashing.  And then several of our cloud platform engineers starting getting BSOD.  Thankfully, we have a mix of Mac and Windows users (with some Linux sprinkled in between).  

Once Crowdstrike removed the offending file, our Windows Dynamic Workers started working shortly after as they were no longer trying to download that file.  We are fortunate as we automatically destroy and recreate our Windows Dynamic Workers.  But our laptops required booting into safe mode and removing the offending file.

What’s scary is both servers and workstations started throwing BSOD around the same time.  In our case, some of the people who were trying to fix the Windows Dynamic Workers couldn’t help initially. 

A phased rollout is a requirement for mission-critical applications. Unfortunately, many believe that means using a load balancer and routing a percentage of traffic. They deploy to a subset of servers or a new set of servers, and they route a percentage of traffic to the new version of the code.

A phased rollout is not feasible or the best solution for many applications. Three canary-style deployment approaches are available.

1. **Traffic-Based:** Deploy to a subset of servers or a new set of servers and route a percentage of traffic to the new version.  Ideal for transactional-style applications or back-end services.  
1. **Feature Toggles:** These deploy to all servers and expose new functionality to a subset of users via a feature flag. They are ideal for applications with user interfaces or when users are required to log in.  
1. **Pilot Customers/Locations:** Deploy the latest code to a subset of locations (such as restaurants, factories, stores, bank branches, etc.)., or to a subset of customers.  It is ideal when the customer/location has dedicated infrastructure.

You can combine feature toggles with traffic-based or pilot-based approaches.  For Octopus Cloud, we opt for a combination of feature toggles and pilot customers.  The customers are selected at random for a specific maintenance window timeframe.  But we might put new functionality behind a feature toggle.  Our new user interface already exists on all our cloud customer instances.  But we are slowly rolling that out using feature toggles.

### There is no such thing as low-risk configuration change

Looking through our Crowdstrike incident channel in Slack, I see many statements, such as:

* What version of the sensor are we running?  
* The sensor version hasn’t changed, why is this happening?  
* We are trying to determine the "good" version: 7.15 or 7.16.  
* The current theory is that the root cause is an out-of-band update.

Those statements are eerily similar to statements I’ve made when debugging after an out-of-band database change is pushed (updated stored procedure, updated view, etc.).  I don’t know what changed, and I’m banging my head against the wall and trying to figure out why I’m getting a new set of errors.  

There is no such thing as a low-risk configuration (or database change).  

Any change that alters the behavior of an application must be treated the same as the application source code.  For any change (source code, feature toggle, configuration file, database schema, etc), it should be easy to know:

* When the change was made  
* What was in the change  
* Why the change was made  
* Who made the change

Debugging without that information is very difficult. It results in a lot of speculation and theories, and debugging is often an act of disqualifying potential suspects. Having information about the who/what/when/why of a change available makes it much easier to debug. The list of potential reasons shrinks drastically.

Ideally all changes would go through the same software delivery process.  But that might not be feasible, for example when using feature toggles.  But any *file* changes \- source code, database schema, configuration files \- must go through the same software delivery process.  

## Conclusion

Think of the takeaways as a defense in depth strategy.  None of them are a magic bullet solution that will prevent bugs from reaching production.  The cumulative impact when combined will reduce the likelihood.  Will bugs still get through?  Of course they will, every software has bugs.  The idea is to create a series of steps to prevent showstopping critical bugs from reaching users and crashing the application.

I hope Crowdstrike’s outage has led to a lot of internal reflection and proposed changes.  However, that reflection should extend beyond Crowdstrike and into the broader IT industry.  It is easy to criticize what went wrong with Crowdstrike, but how many of our applications have similar issues if we are honest with ourselves?  Crowdstrike should be held to a higher standard; however, how many of our applications are mission-critical for our companies?  Are we holding ourselves to the same standard?

Learn from other’s mistakes so we don’t make the same mistakes.  