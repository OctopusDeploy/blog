---
title: Build versus buy
description: Probing the pros and cons of building something yourself versus buying a product
author: shawn.sesna@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - tag
---

The build versus buy dilema is an issue that almost all organizations face.  Whether it's a common scenario or a unique situation, there are often tools available that can address them.  However, there a many factors that need to be considered when making the decision as to whether to build a solution versus buy something that is commercially available.  In this post, we'll explore the considerations that need to be taken into account before making a decision.

## Decision factors
The reasons for building your own solution or purchasing a product usually come down to the following factors: 
- Cost 
- Maintenance
- Support

### Cost
The price of a product can heavily influence an organizations decision as to whether they will purchase.  Small organizations on a shoe-string budget simply may not be able to afford the available solutions and have no other choice then build something that suits their needs.  Larger organizations need to weigh the cost of the solution at scale to make sure it will propertionally deliver value, if increased scale results in an exponential increase in cost, it may not be worth purchasing.  Many paid solutions have gone from the traditional perpetual license to a subscription model.  This means that the cost of the product must now be included in the annual budget.  It is not uncommon for periodic price increases to occur as more features are introduced into the product.  Some increases are made to the product overall versus the increase is only if you want the additional modules.  In the former case, you will need to evaluate whether the product is giving you the same value as it once did.  Complicating this decision is the cost of switching to either a built or competing solution.  The time and energy it will take to onboard people to the new solution, training, hardware purchases, and or additional cloud resources may be more expensive than paying the increased cost.  However, once the switch is complete, the long term savings may be worth it.

Cost does not always involve the purchase of a product, there are costs associated with building your own solution.  In-house built solutions are sometimes considered "free" because it's being developed by people who already work at that organization, these are usually false savings.  While you or your team have the technical ability to build your own solution, the cost to your organization might be more than the commercial product itself.  Salary and benefits are one dimension to consider when determining the true cost of building a custom solution along with time.  The time spent on building and maintaining a solution takes away from other projects that could potentially benefit more to the company than a "free" solution.  That being said, one of the benefits to a home-grown solution is, once complete, there isn't an annual cost that needs to be included other than allocating some amount of time to properly maintain it.  In addition, the solution developed is customized to your organizations needs and contains only the features required whereas a paid solution could potentially contain features that you'd never use.  

### Maintenance
Maintenance of a solution is crucial to keeping it operating effectively.  New versions of Operating Systems (OS) or development frameworks require that any solution be tested to make sure sure it will continue to function.  Companies that provide purchased solutions perform this activity out of necessity to remain in business.  For built solutions, this will require time built into your schedules to adapt it to changes.  One potential pitfall of a built solution is that it doesn't get the attention it requires as it "just works" ... until it doesn't.  In the best cases, this is discovered early and time is allocated to deal with it.  In the worst case, it's discovered at a crucial moment leading to a mad scramble to fix it.

Other than compaitiblity, security is another maintenance concern.  Third-party libraries are often used in both paid and home-built solutions as a way to not reinvent the wheel.  When vulnerabilities are found within third-party libraries, commercial companies react quickly to ensure their software is secure for their customers.  While home-grown solutions may have scanning software built into their Continuous Integration (CI) build process, it only runs when a new version is built.  It may be quite some time before the vulnerability is discovered giving time for it to be exploited.  Source code scanning and runtime vulnerability monitoring tools can help mitigate this risk, but require configuration and maintenance to ensure they are always running.

As previously mentioned, companies who sell solutions are constantly implementing new features to keep up with evolving trends in the industry.  Depending on your needs, this could be a potential benefit as when you decide to follow the evoloving trend, the feature is already there.  A purpose-built, home-grown solution may have to be drastically modified to perform this new task.  For example, deploying an application to IIS is vastly different than deploying to Kubernetes.

### Support
Solutions that have become critical to an organization require support.  It is not uncommon for an organization that has a successful built solution to employ people specifically to support it or support it as part of their job duties.  In fact, successful built solutions sometimes become open-source projects which benefit from crowdsourcing to solve issues that arise.  This relationship can be symbiotic in that those who support the solution can offer help to those who've chosen to implement it into their organization.  However, this also carries a risk in that open-source solutions are sometimes abandoned by their creators or those who originally wrote the solution have moved on, taking their expertise with them.  To highlight this point, Jenkins, a popular CI server, has over [100 plugins](https://plugins.jenkins.io/ui/search/?labels=adopt-this-plugin) listed for adoption.

Conversely, paid products come with support built into their purchase price.  The level of support varies in that there may be additional support levels that can be purchased.  For example, the base product comes with support that is available from 8am to 5pm Monday through Friday, however, the premium support plan carries a 24/7 availability, but costs extra.  Support plan choices can also be dictated by company policy.  Your company may have policy which requires that all commercial products come with 24/7 support.  The obvious benefit to a paid product is that you'll always have someone to to contact should something go wrong.  

## Conclusion
Earlier in my career I wrote my own automated deployment system that I implemented at two different jobs.  One of these organizations is still using the solution today, over a decade since I'd written it.  This demonstrates that built solutions can be quite successful and offer an incredible ROI.  While successful at my second job, my role changed which gave me little time to be able to develop new features much less maintain it.  As time went on, a developer conviced me it was in the organizations best interest to go with a paid product.  After putting my pride aside, I implemented the paid solution, can you guess which one? ;P

While I work for a company that sells software, my goal of this post was to give you things to think about when trying to decide to build a solution or puchase one.  Both options can be beneficial as well as costly, it is up to you to weigh what is best for your organization.