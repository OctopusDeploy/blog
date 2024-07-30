---
title: Lessons from Crowdstrike’s outage
description: Lessons and takeaways from Crowdstrike's outage to apply to our development practices.
author: bob.walker@octopus.com
visibility: public
published: 2024-07-30-1400
metaImage: blogimage-bestpracticecicd-2022.png
bannerImage: blogimage-bestpracticecicd-2022.png
bannerImageAlt: Person with a checklist stands in front of DevOps infinity symbol.
isFeatured: false
tags:
  - DevOps  
---

Many people are now familiar with what happened on Friday, July 19, 2024. A bug in a Crowdstrike configuration file, or "Template Instances," caused the Crowdstrike Falcon Sensor to crash. Unfortunately, the Crowdstrike Falcon Sensor is configured as a Windows kernel boot driver. Because that driver crashed, Windows did what it should when a critical driver crashes: it stopped working and showed the dreaded "blue screen of death".

David Plummer released several videos about the incident:

- [Crowdstrike IT outage explained by a Windows developer](https://www.youtube.com/watch?v=wAzEJxOo1ts)
- The follow-up [Crowdstrike update: Lessons learned from a Microsoft engineer](https://www.youtube.com/watch?v=ZHrayP-Y71Q)
- And he read Crowdstrike’s [preliminary post incident review](https://www.crowdstrike.com/blog/falcon-content-update-preliminary-post-incident-report/) (PIR) 

Watching these videos raises a deeper philosophical discussion about third-party kernel drivers. While perfectly valid, I’m neither expert nor qualified to discuss that topic. 

This post isn't  about what Crowdstrike should have done. Instead, I use the video resources to provide context and takeaways we can apply to our teams and organizations. 

## Background context

The Windows operating system has 2 modes: user and kernel. The kernel performs low-level operating system tasks. This is for good reason, as the access to the kernel is restricted. Most software that needs access to the kernel (or the underlying hardware) often runs in user mode, makes an API call to the kernel, waits for the kernel to act, and then returns the appropriate information. To learn more, please see David Plummer’s [Crowdstrike IT outage explained by a Windows developer](https://www.youtube.com/watch?v=wAzEJxOo1ts) video and the [Architecture of Windows NT](https://en.wikipedia.org/wiki/Architecture\_of\_Windows\_NT) on Wikipedia.

Drivers control the various hardware and can run in user or kernel modes. Crowdstrike’s Falcon Sensor is a kernel driver. For obvious reasons, Microsoft doesn’t let any driver run directly in the kernel without verification. Microsoft runs the updated driver in their labs for testing and verification. Before that, Crowdstrike runs updates through its extensive testing suite. Assuming everything passes, the driver gets signed and released to the public. 

The deployment process of that sensor is quite extensive, as detailed by Crowdstrike in their PIR document.

> This culminates in a staged sensor rollout process that starts with dogfooding internally at CrowdStrike, followed by early adopters. It is then made generally available to customers. Customers then have the option of selecting which parts of their fleet should install the latest sensor release (‘N’), or one version older (‘N-1’) or two versions older (‘N-2’) through Sensor Update Policies.

As David Plummer points out in his videos, getting a driver update through Microsoft’s certification process is not trivial and takes time. Changing a driver alters the digital signature, as both Crowdstrike and Microsoft perform testing. 

With new threats appearing every day, an anti-virus company like Crowdstrike needs to react quickly. Crowdstrike has considered that and can release an update to their definitions and configurations without changing drivers. This is known as "Rapid Response Content".

> Rapid Response Content is used to perform a variety of behavioral pattern-matching operations on the sensor using a highly optimized engine. Rapid Response Content is a representation of fields and values, with associated filtering. This Rapid Response Content is stored in a proprietary binary file that contains configuration data. It is not code or a kernel driver.

> Rapid Response Content is delivered as "Template Instances," which are instantiations of a given Template Type. Each Template Instance maps to specific behaviors for the sensor to observe, detect or prevent. Template Instances have a set of fields that can be configured to match the desired behavior.

> In other words, Template Types represent a sensor capability that enables new telemetry and detection, and their runtime behavior is configured dynamically by the Template Instance (i.e., Rapid Response Content).

> **Rapid Response Content provides visibility and detections on the sensor without requiring sensor code changes.**

The Rapid Response Content gets published to their content configuration system hosted in the cloud. The Falcon sensor (installed locally) downloads those updated files and writes them to disk on the host. 

Any changes and additions to the Rapid Response Content get "stress tested across many aspects, such as resource utilization, system performance impact, and event volume".

## What happened

A malformed Rapid Response Content configuration file made it to the Crowdstrike content delivery system in the cloud. Servers and workstations downloaded that malformed file from the cloud.  The sensor tried to load and run the malformed file, but it encountered a file it couldn’t handle gracefully and crashed.  

Because it was a kernel driver, the entire operating system crashed.  David Plummer stated that this is a feature, not a bug.  It's a safety precaution, as allowing the OS to continue to run with a failed kernel driver could cause more damage. It’d be like continuing to run your car after all the oil leaked out.

The abbreviated timeline is:

- February 28, 2024: Sensor 7.11 gets released, introducing and adding a new mechanism to detect attacks against named pipes.  
- March 5, 2024: The team performed a stress test using that new code and validated it for use.  
- March 5 to April 24, 2024: The team released 4 configuration files for the new functionality to production, and the files performed as expected.  
- July 19, 2024: They deployed 2 more configuration files. One of these files was malformed,  but both passed validation due to a bug in the content validator.  

Interestingly, there were tests to verify the validator (a test of the tests). They felt confident in their tests because they'd successfully deployed other configuration files.

> Based on the testing performed before the initial deployment of the Template Type (on March 05, 2024), trust in the checks performed in the Content Validator, and previous successful IPC Template Instance deployments, these instances were deployed into production.

## Repeating mistakes of the past

Unfortunately, this isn't an isolated incident. Earlier this year, Crowdstrike caused similar [issues with Linux](https://www.theregister.com/2024/07/21/crowdstrike\_linux\_crashes\_restoration\_tools/), specifically Debian-based distros. However, it didn't get much coverage because its impact wasn’t as widespread as the Windows outage. Similar to the Windows outage, testing was a primary cause of the issue.

CrowdStrike acknowledged the issue a day later, but it took weeks to analyze the root cause. The analysis revealed that a specific Debian configuration was missing from their testing matrix.

Similar issues happened a month or so later with Rocky Linux.  

## Future mitigation

In the takeaways section below, I list what we can learn from the Crowdstrike outage. In fairness to Crowdstrike, they listed many mitigation strategies in their PIR. This shows they already know what tests they should have performed and how they should have deployed.  

- Improved testing including local developer testing, content update, and rollback testing, stress testing, and more.  
- Additional validation checks for the content validator (more testing for the testing tool). 
- Improved error handling in the content interpreter.  
- Improved deployments including a canary style deployment, improved monitoring during that rollout, and providing customers greater control over the delivery of the Rapid Response Content.
- Third-Party validation including independent third-party security code reviews and end-to-end processes.

## Takeaways

The outage on July 19, 2024, was [a disaster](https://www.merriam-webster.com/dictionary/disaster). Outages to 911 and global ground stops for major airlines are very serious. Like many disasters, there isn’t one specific thing we can point to, it's a combination of issues. And like many disasters, there wasn't malicious intent. No one at Crowdstrike woke up on July 19 and said, "I hope I can bring the earth to a standstill today."

I'm not privy to Crowdstrike's internal programming standards, SDLC, and CI/CD processes. As such, I don’t think it is fair for me to comment on what they should or shouldn’t change internally. 

However, we can learn from their mistakes and reflect on changes we can make in our teams and organizations.

### Input validation for edge cases is critical

In Crowdstrike’s PIR, they mention needing to "Enhance existing error handling in the Content Interpreter."  Earlier in the PIR, they mention, "The Content Interpreter is designed to gracefully handle exceptions from potentially problematic content."  They have input validation in place but were missing edge cases.

Input validation is important. However, there will always be edge cases.  [Injection](https://owasp.org/Top10/A03\_2021-Injection/) is number 3 on OWASP’s top 10 list for a reason. Often, it's a failure of imagination or something that seems so unlikely it doesn’t merit a test. If you're writing input validation, have an attacker mindset. Try to break your system.  Have others on your team do the same.

### Test with real-world data

Crowdstrike’s PIR provides some detail on how they test these configuration updates.  

> Newly released Template Types are stress tested across many aspects, such as resource utilization, system performance impact and event volume. For each Template Type, a specific Template Instance is used to stress test the Template Type by matching against any possible value of the associated data fields to identify adverse system interactions.

These configuration files help detect new attack vectors and viruses.  Interestingly, they don’t explicitly state they write an attack specifically for the sensor to detect on a Windows server or workstation.

People spend lots of time writing unit and integration tests and configuring the external data to generate a repeatable result. For unit tests, we use fakes and mocks. Data gets injected into a database or written to a file system for integration tests for consistency.  That consistency prevents the dreaded "flaky" test.  

However, that data is only as good as the imagination of the person who created it.  Nothing can replace real-world use.  Remember, users are the ultimate bringers of chaos.

While working at DTN, I had to rewrite a critical command processor. External sites sent commands following the PIDX protocol to our servers. The command processor read the database and determined whether to send a confirmation or denied message.  When I finished the new version, it had many unit and integration tests covering all the possible scenarios our QA team and I could think of.  However, the persistent fear was that it would fail when we switched from the legacy system to the new one.  So, I wrote a testing application called the "chaos monkey" that read the commands from the previous week, sent them through the legacy and new systems, and compared the responses.  Any mismatch in the responses got logged for analysis.  The first time I turned it on, I found configurations I hadn’t dreamed of.  

This won't work for all applications, but there are alternatives. When working at Farm Credit, we'd refresh our staging environments with sanitized data from production. Our DBAs and web admins created an entire workflow to copy production to staging and remove the necessary PII, so we had real-world data to test before production.  

Putting your application through testing scenarios using real-world data will help you find bugs before they reach production.  While this may not be feasible for every application, it's worth considering.

### Dogfood when feasible

Another interesting point from Crowdstrike’s PIR is the difference between pushing a sensor update and Rapid Response Content.

> The sensor release process begins with automated testing, both prior to and after merging into our code base. This includes unit testing, integration testing, performance testing and stress testing. This culminates in a staged sensor rollout process that starts with ***dogfooding internally*** (my emphasis) at CrowdStrike, followed by early adopters. It is then made generally available to customers. Customers then have the option of selecting which parts of their fleet should install the latest sensor release (‘N’), or one version older (‘N-1’) or two versions older (‘N-2’) through Sensor Update Policies.

> Template Instances are created and configured through the use of the Content Configuration System, which includes the Content Validator that performs validation checks on the content before it is published.

It appears Crowdstrike dogfoods the sensor updates but not the Rapid Response Content update. 

Dogfooding is when you use your application every day. For example, we use Octopus to deploy Octopus Deploy. The instance we use (deploy) gets updated daily (sometimes many times) using the latest build. We also have several employee instances that get updated before our customers. This helps us find showstopping bugs before they reach the public. 

Whenever possible, you should dogfood your applications. You'll find showstopping bugs, annoyances, quirks, and other miscellaneous bugs that automated tests might miss.

### Adopt canary or phased rollouts for mission-critical applications

After a Rapid Response Content file is available in Crowdstrike’s Cloud, clients download it.  

Our director of security said his laptop suddenly went to the black screen of death (BSOD) around 2:30pm Friday, Australian time. He rebooted the machine, and again experienced the BSOD.  Around that time, our cloud platform engineers noticed our Windows dynamic workers crashing.  Then, several of our cloud platform engineers starting seeing the BSOD.  Thankfully, we have a mix of Mac and Windows users (with some Linux sprinkled between).  

After Crowdstrike removed the offending file, and thanks to the quick actions of our Cloud Platform Team's incident response, our Windows dynamic workers started working.  We're fortunate as we automatically destroy and recreate our Windows dynamic workers.  But we needed to boot our laptops into safe mode and remove the offending file.

What’s scary is both servers and workstations started throwing BSOD around the same time.  In our case, some of the people who were trying to fix the Windows dynamic workers couldn’t help initially. 

A phased rollout is a requirement for mission-critical applications. Unfortunately, many believe that means using a load balancer and routing a percentage of traffic. They deploy to a subset of servers or a new set of servers, and they route a percentage of traffic to the new version of the code.

A phased rollout is not feasible or the best solution for many applications. Three canary-style deployment approaches are available.

1. Traffic-based: Deploy to a subset of servers or a new set of servers and route a percentage of traffic to the new version.  This is ideal for transactional-style applications or back-end services.  
2. Feature toggles: These deploy to all servers and expose new functionality to a subset of users via a feature flag. They're ideal for applications with user interfaces or when users need to log in.  
3. Pilot customers/locations: Deploy the latest code to a subset of locations (like restaurants, factories, stores, bank branches) or to a subset of customers.  This is ideal when the customer/location has dedicated infrastructure.

You can combine feature toggles with traffic-based or pilot-based approaches.  For Octopus Cloud, we use a combination of feature toggles and pilot customers.  The customers get selected at random for a specific maintenance window timeframe.  But, we might put new functionality behind a feature toggle. Our new user interface already exists on all our cloud customer instances. But we're slowly rolling that out using feature toggles.

### There's no such thing as low-risk configuration changes

Looking through our Crowdstrike incident channel in Slack, I see many comments and questions, like:

- What version of the sensor are we running?  
- The sensor version hasn’t changed, why is this happening?  
- We're trying to determine the "good" version: 7.15 or 7.16.  
- The current theory is that the root cause is an out-of-band update.

These statements are eerily similar to statements I’ve made when debugging after an out-of-band database change gets pushed, like an updated stored procedure, updated view, etc. I don’t know what changed, and I’m struggling to figure out why I’m getting a new set of errors.  

There's no such thing as a low-risk configuration (or database) change.  

If a change alters the behavior of an application, you must treat it the same as the application source code. For any change, like source code, feature toggle, configuration file, database schema, it should be easy to find out:

- When the change was made  
- What was in the change  
- Why the change was made  
- Who made the change

Debugging without that information is very difficult. It results in a lot of speculation and theories, and debugging is often about disqualifying potential suspects. Having information about the who, what, when, and why of a change makes it much easier to debug. The list of potential reasons shrinks drastically.

Ideally, all changes should go through the same software delivery process. But that might not be feasible, for example when using feature toggles.  But any *file* changes – source code, database schema, configuration files – must go through the same software delivery process. 

## Conclusion

Think of the takeaways in this post as a defense strategy. There's no single solution that will prevent bugs from reaching production, but the cumulative impact will reduce the likelihood. Bugs will still get through. Every software has bugs. The aim is to create a series of steps to prevent showstopping critical bugs from reaching users and crashing the application.

I hope Crowdstrike’s outage has led to a lot of internal reflection and proposals for change. That reflection should extend beyond Crowdstrike and into the broader IT industry. It's easy to criticize what went wrong, but how many of our applications have similar issues if we're honest with ourselves? Crowdstrike should be held to a higher standard; however, many of our applications are mission-critical for our organizations, too. Are we holding ourselves to the same standard?

We want to learn from each other’s mistakes, so we don’t make the same ones.

Happy deployments!