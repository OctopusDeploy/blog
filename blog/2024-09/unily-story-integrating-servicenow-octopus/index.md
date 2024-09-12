---
title: "Integrating ServiceNow and Octopus to increase efficiency: Unily’s story"
description: Unily uses Octopus and ServiceNow to manage complex deployments to all customers. In this post, we dive into Unily’s solution and how the integration helps minimize downtime and increase efficiency.
author: ella.pradella@octopus.com
visibility: public
published: 2024-09-23-1400
metaImage: blog-unily-casestudy-2024.png
bannerImage: blog-unily-casestudy-2024.png
bannerImageAlt: Person holding a giant magnifying glass over the Unily logo on a giant piece of paper.
isFeatured: false
tags: 
  - DevOps
  - Product
  - Integrations
---

Many organizations use ServiceNow to automate change management, but you can also simplify change management by integrating ServiceNow with Octopus’s IT Service Management (ITSM) support.

The Unily team use Octopus and ServiceNow to manage complex deployments to all customers. In this post, we dive into Unily’s solution and how the integration helps minimize downtime and increase efficiency.

## How Unily uses ServiceNow

Unily provides a business-critical service to its customers with a true enterprise EX platform. One of Unily's unique challenges is managing a complex deployment strategy for its software updates. This task is particularly intricate given the diverse needs of deployments.

You must run many deployments and maintenance tasks at times convenient for your customers, but you must also offer a great experience to schedule and maintain those tasks. How do you achieve flexibility and a great customer experience without dropping quality or blowing up your budget by hiring many engineers?

Unily solved the problem via expert-level automation. They combined two best-in-breed customer service and deployment tools: ServiceNow and Octopus Deploy.

ServiceNow gave Unily a self-service customer portal and automated the service and change management process. Octopus Deploy enabled automated, flexible deployments and maintenance tasks at scale.

## Integrating ServiceNow with Octopus

By integrating ServiceNow with Octopus, the Unily team automated their deployments. Unily also created an internal tool that automates deployments and maintenance tasks for its engineers. When a deployment gets scheduled in ServiceNow, Unily’s custom automation-platform runs an automated deployment or assigns a task to the engineer, should they need to act manually by creating an internal Incident in ServiceNow along with the deployment details from Octopus Deploy.

For manual deployments, the engineer adds the ServiceNow ID to Octopus to confirm the client, time, and deployment type. If any details don’t match, the deployment won’t proceed, assuring deployment compliance and reducing risk. Engineers can also schedule deployments and set the maximum time for the deployment to run, receiving notifications if there are problems.

When the deployment finishes, Octopus pushes the start and end time to ServiceNow and marks it complete. Along with the automation, the deployment that's triggered in ServiceNow also notifies clients when a deployment starts, and when the deployment completes.

Unily also uses its custom automation platform with ServiceNow and Octopus to run various automated tasks using the API, including searching for expired certificates in Octopus variables. Unily notifies clients 2 months in advance of an upcoming certificate expiration. This is done as an email asking them to refresh the certificate. This workflow integrates with ServiceNow to ensure all information is in one place. 

## The benefits of integrating Octopus and ServiceNow for Unily

Integrating Octopus and ServiceNow helped Unily achieve the following benefits:

- Engineers can use process automation to focus on other tasks.
- A 10x increase in deployments without growing the team.
- More deployment slots are made available.
- Ensured compliance and reduced risk thanks to guardrails that prevent deployments without the appropriate ticket.
- Simplified maintenance through automated tasks.

From September 2020, when the dedicated deployment team was first established at Unily, the team went from doing 200 deployments per month to almost 3,000 deployments. Along with increased capacity, Unily also reduced the failure rates dramatically, only having to deal with a failure rate of 1.2% where an engineer may need to step in.

## Conclusion

Unily has a complex use case involving deployments to customers with many different needs. 

By integrating Octopus with ServiceNow, Unily automates its deployments and maintenance tasks, increases its deployment frequency, maintains customer flexibility, and reduces risk. 

You can read more about [Unily’s story in our case study](https://octopus.com/company/customers/casestudies/unily).

[ServiceNow](https://octopus.com/docs/approvals/servicenow) and [Jira ITSM](https://octopus.com/docs/approvals/jira-service-management) integrations are available to all enterprise customers.

Happy deployments!