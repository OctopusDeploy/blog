AI has generated a lot of excitement, and criticism, ever since ChatGPT highlighted the potential and limitations of generative AI. DevOps teams have a unique challenge developing and deploying AI platforms because they are the ones tasked with realizing the value of AI and delivering it to customers in a safe, reliable, and predictable manner.

Fortunately, many of the best practices that we have adopted for deploying and maintaining software still apply to those responsible for AI platforms. Some practices, like testing, have a renewed sense of urgency given the nondeterministic nature of GenAI. Some, like rollbacks, may end up being easier thanks to the stateless design of GenAI models.

AI teams may also have the luxury of working on greenfield projects, which provides them with the opportunity to design their DevOps processes from the ground up, taking advantage of trends like platform engineering and focusing on DevEx.

In this post we’ll explore the practices, ideals, and compromises that DevOps teams spinning up new AI projects must take into account as they lay the foundation of their DevOps lifecycle.

## It starts and ends with DevEx

DevOps started with an insight as obvious as it was brilliant: what if instead of separate development and operations teams with distinct management hierarchies, measured on (often competing) outcomes, and communicating through tickets, we had one team reporting to a shared management hierarchy measured on shared outcomes focused on delivering value to customers?

Over the years though DevOps has come to encompass so much that it has almost no meaning. To say you have a DevOps job has about as much meaning as saying you have a sales, finance, or management job. It is the nature of DevOps to consume as much responsibility as it can (demonstrated by all the  Dev-Insert-Something-Here-Ops paradigms that have evolved), often without any consideration for who has to deliver this growing sphere of responsibility. The frustration by DevOps team members with the inability to answer the question “What am I not responsible for?” has led to the rise of DevEx.

As noted in the paper [DevEx: What Actually Drives Productivity](https://queue.acm.org/detail.cfm?id=3595878), DevEx has three dimensions: flow state, cognitive load, and feedback loops:

![](devex.png)

DevOps teams can support all three of these dimensions by ensuring every member of the team can confidently answer the question “What am I not responsible for?” Keeping this question in mind as you develop your DevOps processes is a powerful strategy to take the best that DevOps has to offer without inadvertently building a system that only functions because everyone is responsible for everything all the time.

## DevEx as a Service

Platform engineering is one of the most effective ways to answer the question “What am I not responsible for?” At its core, platform engineering, and specifically the Internal Developer Platform (IDP) that is the interface between the platform and DevOps teams, must satisfy three requirements:

1. Provide a repository of architectural decisions.
2. Enable architectural decisions to be implemented at scale.
3. Define feedback processes that ensure architectural decisions are updated over time.

![](idp.png)

In this context, we refer to the book [Objects, Components, and Frameworks With UML: The Catalysis Approach](https://www.amazon.com/Objects-Components-Frameworks-UML-Catalysis/dp/0201310120) by Desmond D'Souza and Alan Wills for this definition of “architecture”:

> The set of design decisions about any system (or smaller component) that keeps its implementors and maintainers from exercising needless creativity.

In other words, architectural decisions answer the question “What am I not responsible for?” by providing DevOps teams with golden pipelines, common tools, processes, and best practices used as the foundation upon which to build valuable solutions to meaningful problems.

When the goal of platform engineering is to deliver improved DevEx, the end result is [DevEx as a Service (DEaaS)](https://octopus.com/publications/devex-as-a-service).

The architectural decisions maintained by your DEaaS implementation will adopt one of three responsibility models.

**Customer responsibility** (or eventual inconsistency), where a customer takes a copy of an artifact capturing architectural decisions and then owns it. 

![](customer-responsibility-model.png)

**Shared responsibility** (or eventual consistency), where both the DEaaS team and customers collaborate on shared artifacts. 

![](shared-responsibility-model.png)

**Central responsibility** (or enforced consistency), where the DEaaS team owns the artifacts and expose controlled interfaces to customers.

![](central-responsibility-model.png)

Bringing this back to the question “What am I not responsible for?”:

* The customer responsibility model makes the DEaaS team responsible for creating artifacts, while customers are responsible for editing and maintaining artifacts after they take ownership.
* The shared responsibility model makes the DEaaS team responsible for creating artifacts and providing a process through which customers can contribute updates, while customers have the responsibility (and sometimes the obligation) to contribute improvements to the artifacts.
* The central responsibility model makes the DEaaS team responsible for creating and maintaining artifacts. Customers can use the artifacts, but are not responsible for maintaining them.

The responsibility models are each subject to constraints around who can edit artifacts, how artifacts are maintained over time, and how many artifacts can be managed by the DEaaS team. These constraints are captured by the responsibility triad, where artifacts maintained by the DEaaS team can optimize for any two of the three concerns:

![](optimize-for-any-two.png)

When implemented correctly, every member of the DevOps team can clearly identify what they are, and are not, responsible for when consuming artifacts provided by the DEaaS team. This allows them to focus on building valuable solutions to meaningful problems.

## The 10 pillars of pragmatic deployments

There are a number of common non-functional requirements associated with the DevOps lifecycle that DEaaS teams must consider as they decide which architectural decisions to share with the DevOps teams. These have been grouped into the 10 pillars of pragmatic deployments.

AI developers are held to a high standard, with AWS noting that governance, defined as “Incorporating best practices into the AI supply chain, including providers and deployers”, is a [core dimension of responsible AI](https://aws.amazon.com/ai/responsible-ai/).

**Repeatable deployments** ensure that DevOps teams can deploy new features and fixes in an automated and consistent manner. While it may be necessary to involve some human decision-making before software is deployed to production, the low level work involved in deployments must be automated. [Google's AI and ML perspective: Operational excellence](https://cloud.google.com/architecture/framework/perspectives/ai-ml/operational-excellence) documentation notes that "Automation enables seamless, repeatable, and error-free model development and deployment."

**Verifiable deployments** bake testing into the deployment process. Testing has added significance when deploying AI applications because GenAI models are non-deterministic by design, which means every deployment is effectively slightly broken all the time. Tests are crucial for ensuring that AI applications continue to meet minimum requirements while catching the kind of subtle and hard to diagnose bugs that can arise from seemingly simple changes to underlying models. Google calls this out with “Test, Test, Test” as part of their [responsible AI practices](https://ai.google/responsibility/responsible-ai-practices/), saying developers should “Conduct integration tests to understand how individual ML components interact with other parts of the overall system.”

**Seamless deployments** implement strategies like blue/green and canary deployments to enable progressive delivery and facilitate quick recoveries. Much like verifiable deployments, seamless deployments are important for AI applications because it can often be difficult to fully understand the outcome of every model change. Rolling changes out slowly or having the ability to quickly revert a deployment can reduce the risk of unintended changes negatively affecting your customers. [Google's AI and ML perspective: Operational excellence](https://cloud.google.com/architecture/framework/perspectives/ai-ml/operational-excellence) documentation advises teams to "Implement phased release approaches such as canary deployments or A/B testing, for safe and controlled model releases."

**Recoverable deployments** are related to seamless deployments, and focus on how quickly a team can roll forward or backward to restore a production service. Fortunately, AI platforms don’t tend to have a large reliance on persistent state, meaning deployments can be easily rolled back. The [Achieve operational excellence with well-architected generative AI solutions using Amazon Bedrock](https://aws.amazon.com/blogs/machine-learning/achieve-operational-excellence-with-well-architected-generative-ai-solutions-using-amazon-bedrock/) post notes that "Automated deployment techniques together with smaller, incremental changes reduces the blast radius and allows for faster reversal when failures occur."

**Visible deployments** allow DevOps teams to answer questions like “What is the state of production?” and “What changed since the last deployment?” [Microsoft’s Human AI Experience (HAX) workbook](https://www.microsoft.com/en-us/haxtoolkit/workbook/) lists “Notify users about major changes” as one of the guidelines, which requires a good understanding of how the production environment has changed between deployments.

**Measurable deployments** are essential to measuring the performance of a DevOps team against common metrics like DORA metrics. Capturing these metrics in the deployment process ensures they are consistent and reliable.

**Auditable deployments** allow DevOps teams to track what changes were made, who made the change, and when the change was made. CSIRO calls out “supply chain accountability” as a core part of [their responsible AI research](https://www.csiro.au/en/research/technology-space/ai/responsible-ai).

**Standardized deployments**, or golden pipelines, are a common type or architectural decision shared by the DEaaS team. Golden pipelines define common deployment steps and encapsulate business requirements like security scanning, manual approvals, deployment windows, and notifications. Modeling deployments and resoutces using Infrastructure as Code (IaC) allows them to be recreated at scale. [Google's AI and ML perspective: Operational excellence](https://cloud.google.com/architecture/framework/perspectives/ai-ml/operational-excellence) documentation says "Manage your infrastructure as code (IaC). This approach enables efficient version control, quick rollbacks when necessary, and repeatable deployments."

**Maintainable deployments** automate day 2 maintenance and ad-hoc tasks like getting log files, restarting services, performing backups, or applying updates. Teams responsible for managed AI services will also benefit from automating processes like adjusting token limits.

**Coordinated deployments** allow DevOps teams to synchronize the deployment of two or more applications that are coupled to each other. They also expose business processes such as approvals with ITSM platforms to ensure deployments are performed in conjunction with other business units and all appropriate stake holders have given their approval. [Microsoft’s Responsible AI principles and approach](https://www.microsoft.com/en-us/ai/principles-and-approach) calls out accountability as a key requirement, asking “How can we create oversight so that humans can be accountable and in control?”

The 10 pillars of pragmatic deployments each represent architectural decisions to bake into the DevOps lifecycle. Understanding which of them are important to your team and how they support AI best practices removes a manual decision your DevOps team needs to make as they deliver new features to your customers.

## Summary

It is perhaps not surprising to discover that AI and DevOps best practices have a significant overlap. All the major cloud providers emphasize testing, supply chain accountability, human oversight, and transparency, all of which are supported by a robust DevOps lifecycle.

Consistently delivering a high quality product means best practices can not be opt-in. By defining best practices as architectural decisions that are implemented at scale throughout your DevOps teams and refined over time allows DevOps teams to focus on the creative and rewarding task of creating valuable solutions to meaningful problems rather than being burdened with menial tasks.

DEaaS provides a framework for thinking about how these architectural decisions allow DevOps team members to answer the question “What am I not responsible for?”, which in turn supports good DevEx, with the 10 pillars of pragmatic deployments listing the non-functional requirements found in a robust DevOps lifecycle.


