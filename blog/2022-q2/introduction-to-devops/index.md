---
title: An introduction to DevOps
description: We take surface-level look at the concepts, tools and roles of DevOps, plus how Octopus fits in.
author: andrew.corrigan@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: blogimage-placeholder.png
bannerImage: blogimage-placeholder.png
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - DevOps
  - Runbooks Series
  - Runbooks
---

'DevOps' is a *big* term, wide-reaching and applicable to many things that surround and support software development. 

For the uninitiated, DevOps can be hard to understand. Is it a culture? A guideline for development processes? A set of tools? Someone's job?

The answer to all these questions is yes! It's all the above.

In this blog, we take a surface-level look at the parts that make up DevOps. We explore the concepts, tools, unique roles, plus explain where Octopus fits in.

## DevOps as a concept

In modern development, it's more important than ever to deliver software and updates as quickly and reliably as possible.

The concept behind DevOps, then, is about removing barriers that get in the way of software delivery.

Typical barriers could include:

- Manual processes
- Office politics
- Counteractive support workflows

The reason to overcome these barriers is simple: If the product is software, then all processes should work towards delivering that software, not against it.

Let's look at what we must consider when thinking about a DevOps approach.

## DevOps as a culture

In traditional organizations, developers and those that support them (such as operations or database teams) are often separate entities with little interaction.

This means each team has its own beliefs, responsibilities, and priorities. While not exactly at odds, often those priorities can clash in pursuit of doing best for the business.

Embracing DevOps as a culture means to remove that friction by introducing a shared purpose.

The methods to achieve that can differ and depend how entrenched people are in their way of working. The aim, however, is to ensure all teams:

- Build trust through clear, honest communication and feedback
- Collaborate throughout the product's entire lifecycle
- Have what they need to make quick decisions
- Can take risks without fear of failure or blame
- Are collectively responsible for the product's success
- Review what worked and what didn't for the next lifecycle

## Why automation is important for DevOps

In DevOps, if you can automate it, you should automate it.

The benefits of automation are simple:

- Teams can focus more on the product
- Results are repeatable and predictable
- Teams can respond much quicker and flexibly to problems

With that, it's no surprise that Continuous Integration and Continuous Deployment (CI/CD) are such a huge part of DevOps.

## DevOps in action: different approaches to DevOps

Given DevOps' conceptual nature, there are many ways for organizations to approach its adoption.

Octopus's own Alex Yates highlighted 2 approaches worth revisiting in his piece [On the naming of "DevOps Engineers"](https://octopus.com/blog/on-the-naming-of-devops-engineers).

### CALMS (Culture, Automation, Lean, Measurement, Sharing)

CALMS is a framework explored in [The DevOps Handbook](https://www.amazon.com.au/Devops-Handbook-World-Class-Reliability-Organizations/dp/1950508404/ref=sr_1_1?crid=22X11LJN7ZYVQ&keywords=the+devops+handbook&qid=1643682944&sprefix=the+devops+handbook%2Caps%2C265&sr=8-1).

CALMS is an acronym where each letter describes the actions needed to adopt DevOps"

- **Culture** - Remove silos and share responsibility
- **Automation** - Reduce time spent on manual tasks
- **Lean** - Streamline processes to reduce wasted time
- **Measurement** - Collect and review data to find areas for improvement
- **Sharing** - Open and honest collaboration between teams

CALMS is also the approach Atlassian took on its path to DevOps culture, using it to measure progress and success. You can read [how Atlassian uses CALMS on their DevOps site](https://www.atlassian.com/devops/frameworks/calms-framework).

### The Three Ways (Flow, feedback, continual experimentation and learning)

Featured in both [The DevOps Handbook](https://www.amazon.com.au/Devops-Handbook-World-Class-Reliability-Organizations/dp/1950508404/ref=sr_1_1?crid=22X11LJN7ZYVQ&keywords=the+devops+handbook&qid=1643682944&sprefix=the+devops+handbook%2Caps%2C265&sr=8-1) and DevOps novel [The Unicorn Project](https://www.amazon.com.au/Phoenix-Project-Devops-Helping-Business/dp/1942788290/ref=sr_1_1?keywords=the+pheonix+project+book&qid=1643683007&sprefix=The+pheonix+pro%2Caps%2C253&sr=8-1), The Three Ways boils DevOps down to 3 key principles:

- Flow
- Feedback
- Continual experimentation and learning

Let's take a quick look at what these principles mean.

#### The First Way: Flow

'The First Way' is about refining every process that takes place between the developer and the customer.

This means:

- Not trying too much during one lifecycle by focusing on short sprints
- Removing arbitrary processes and giving the team what they need to keep things moving

#### The Second Way: Feedback loops

'The Second Way' is all about faster feedback.

Faster feedback means faster reactions, so you can:

- Address problems before they become bigger problems deeply rooted due to iteration
- Troubleshoot with less reverse engineering
- Adjust processes to better meet the needs of customers and your teams

#### The Third Way: Experiment and learn

'The Third Way' is about recognizing that it's okay to take risks and that failure is an important part of learning. It's also about communicating the wins too.

By recognizing this, it allows your teams to:

- Think outside the box
- Learn from mistakes
- Test the resilience of your DevOps culture

## DevOps roles

Though the concept is about a unifying purpose, DevOps is really everyone's role. That said, it's still important for everyone to know their responsibilities. Adopting DevOps means adding some specialist roles that sit alongside development staples like coders, QA, designers, and more.

Let's take a quick look at some of the common extra roles needed in DevOps and what they do. Some of the naming conventions and finer responsibilities may differ between organizations.

### DevOps Engineer

The exact function of DevOps Engineers is often debated. The consensus, though, is that the role exists so everyone else can perform theirs as smoothly as possible.

Both conductor and problem solver, a DevOps Engineer oversees the bigger picture, managing:

- Workflows
- Infrastructure setup and config (including scaling)
- System permissions
- Needs of various teams

### Build Manager

The Build Manager maintains the automation systems that make up Continuous Integration (CI).

This means ensuring code gets compiled, built, tested and handed off for deployment.

### Release Manager

A Release Manager directs builds promoted for release. They communicate what's included in a release and plots their course through a pipeline's environments.

This makes up what we call Continuous Deployment or Continuous Delivery (CD) - the process we created Octopus to help with.

### Product Manager

Product Managers are almost a conduit between developers and end users.

Working with developers, they help ensure the product will have the features and fixes customers need.

### Data Analyst

This one's simple. A Data Analyst scours data to spot patterns and find areas for improvement.

This helps you spot the things that impact user experience, such as feature problems or product navigation.

## Example DevOps tools

Most DevOps approaches agree a product lifecycle should look something like this:

1. Plan
2. Code
3. Build
4. Test
5. Package
6. Release
7. Deploy
8. Operate
9. Monitor

Rinse and repeat!

It's important to find the right tools to help you manage each phase. Let's look at a few examples.

### Planning

The planning stages are the most important for the direction and future of your product. Effective planning helps you focus on the improvements and features users will get at the end of the next lifecycle.

Given a lot of that process can get pretty conceptual, there's a range of tools that allow collaboration and project management.

[Slack](https://slack.com/intl/en-au/) and [ZOOM](https://zoom.us/) are our main tools at Octopus, but other popular options include:

- [Teams](https://www.microsoft.com/en-au/microsoft-teams/group-chat-software)
- [Confluence](https://www.atlassian.com/software/confluence)
- [Monday.com](https://monday.com/)
- [Trello](https://trello.com/)

### Code repositories and source control

Everyone knows source control is important to any development team. Source control tracks and checks every new piece of code and file change.

Most developers use [Git](https://git-scm.com/) for source control. Git is both a system and philosophy, allowing for distributed development and helps avoid risk through branching.

Code repositories, then, are hosting services for Git-managed code.

Popular options include:

- [GitHub](https://github.com/)
- [GitLab](https://about.gitlab.com/)
- [Bitbucket](https://bitbucket.org/)

### Build and test

Build servers (also known as CI platforms) can save time by automating:

- code compiling
- code-validation tests
- package creation.

Popular options include:

- [Jenkins](https://www.jenkins.io/)
- [GitHub Actions](https://github.com/features/actions)
- [TeamCity](https://www.jetbrains.com/teamcity/)
- [Circle CI](https://circleci.com/)
- [Atlassian Bamboo](https://www.atlassian.com/software/bamboo)
- [Azure DevOps](https://azure.microsoft.com/)

We spent the first quarter of 2022 doing deep dives into [Jenkins and GitHub Actions](https://octopus.com/blog/introduction-to-build-servers). Why not take a read and see if they're for you?

### Package

Software packaging tools turn your code into the deployable artifacts. You host and deploy these artifacts from package repositories.

Popular options include:

- [Docker Hub](https://www.docker.com/)
- [JFrog](https://jfrog.com/)
- [Codefresh](https://codefresh.io/)
- [Redhat Quay](https://www.redhat.com/en/technologies/cloud-computing/quay)

### Releases and deployments

Most code repos and build servers allow you to manage releases and deploy to targets in some fashion. That said, they don't solve the same problems dedicated release management or deployment tools do.

Obviously, [Octopus](https://octopus.com/) is our deployment tool and we think it's pretty great. We'll talk about [where Octopus fits into DevOps](#octopus-and-devops) later.

Other popular options include:

- [Atlassian Bamboo](https://www.atlassian.com/software/bamboo)
- [AWS CodeDeploy](https://aws.amazon.com/codedeploy/)
- [GitLab CI/CD](https://about.gitlab.com/)
- [DeployHQ](https://www.deployhq.com/)
- [ElectricFlow](https://digital.ai/technology/electricflow)

### Operations

Operations relates to the setup, running and maintenance of infrastructure for a pipeline.

Popular options include:

- [Saltstack](https://saltproject.io/)
- [Chef](https://www.chef.io/)
- [Puppet Enterprise](https://puppet.com/try-puppet/puppet-enterprise/)

[Runbooks](#octopus-and-devops) are also an important feature related to operations. We talk a little more about Octopus Runbooks at the end.

### Monitoring

Monitoring tools scrape your product and related systems for important data. This can inform decisions about performance and customer usage.

Popular options include:

- [New Relic](https://newrelic.com/)
- [Splunk](https://www.splunk.com/)
- [Dynatrace](https://www.dynatrace.com/)
- [Datadog](https://www.datadoghq.com/)

## Where Octopus fits into DevOps {#octopus-and-devops}
 
Octopus fits nicely into a DevOps environment in 2 key ways:

1. Providing the Continuous Deployment in your CI/CD pipeline and making complex deployments simple.
2. Octopus Runbooks - Runbooks allow you to automate routine or emergency operations tasks. This could include:
   - Incident recovery
   - Backups, restores, and tests
   - Spin-up and tear-down of infrastructure
   - The stop, start and restart of system services
   - File clean-up
   - The running of scripts in any language you need

We're looking closely at Octopus Runbooks in the next few months, with detailed insight, guides, and samples.

Happy deployments!