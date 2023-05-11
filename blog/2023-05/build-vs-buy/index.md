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
The price of a product can heavily influence an organization as to whether they will purchase.  Small organizations on a shoe-string budget simply may not be able to afford the available solutions and have no other choice then build something that suits their needs.  Larger organizations need to weigh the cost of the solution at scale to make sure it will propertionally deliver value, if increased scale results in an exponential increase in cost, it may not be worth purchasing.

Cost does not always involve the purchase of a product, there are costs associated with building your own solution.  In-house built solutions are sometimes considered "free" because it's being developed by people who already work at that organization.  While you or your team have the technical ability to build your own solution, the cost to your organization might be more than the commercial solution itself.  Salary and benefits are one dimension to consider when determining the true cost of building a custom solution along with time.  The time spent on building and maintaining a solution takes away from other projects that could potentially benefit more to the company than a "free" solution.

### Maintenance
Maintenance of a solution is crucial to keeping it operating effectively.  New versions of Operating Systems (OS) or development frameworks require that any solution be tested to make sure sure it will continue to function.  Companies that provide purchased solutions perform this activity out of necessity to remain in business.  For built solutions, this will require time built into your schedules to properly maintain the solution.  One potential pitfall of a built solution is that it doesn't get the attention it requires as it "just works" ... until it doesn't.

Other than compaitiblity, security is another maintenance concern.  Third-party libraries are often used in both paid and home-built solutions as a way to not reinvent the wheel.  When vulnerabilities are found within third-party libraries, commercial companies react quickly to ensure their software is secure for their customers.  While home-grown solutions may have scanning software built into their Continuous Integration (CI) build process, it only runs when a new version is built.  It may be quite some time before the vulnerability is discovered.

### Support
Solutions that have become critical to an organization require some level of support.  It is not uncommon for an organization that has a successful built solution to employ people to support it.  In fact, successful built solutions sometimes become open-source projects which benefit crowdsourcing to solve issues that arise within the solution.  This relationship can be symbiotic in that the people who support the solution for their organization can offer support to those who've chosen to implement that solution into their own organization.  That being said, built solutions sometimes carry the risk of those who originally authored the solution have moved on, leaving little to no support should something go wrong.

Conversely, paid products come with support built into their purchase price.  The level of support the customer receives may vary in that there may be additional support levels that can be purchased.  For example, the base product comes with support that is available from 8am to 5pm Monday through Friday, however, the premium support plan carries a 24/7 availability but costs extra.  In addition, many software companies have changed their pricing to fall within a subscription model which includes access to their support.  This means a paid product becomes an annual expense.

## Conclusion
Earlier in my career I wrote my own automated deployment system that I implemented at two different jobs.  One of those organizations is still using the solution today, over a decade since I'd written it.  This demonstrates that built solutions can be quite successful and offer an incredible ROI.  While successful at my second job, my role changed which gave me less and less time to be able to develop new features much less maintain it.  As time went on, a developer conviced me it was in the organizations best interest to go with a purpose-built product.  After putting my pride aside, I implemented a paid solution, can you guess which one? ;P

While I work for a company that sells software, my goal of this post was to give you things to think about when trying to decide to build a solution or puchase one.  Both options can be beneficial as well as costly, it is up to you to weigh what is best for your organization.


## Body

The body of the post is where you share your hypothesis, how-to, or story.

If there are any previous posts on the same topic, please link to them to help with our SEO efforts. For example:
Our post about [DORA metrics](https://octopus.com/blog/dora-metrics-devops-business-outcomes) discusses how agility-based metrics can help improve profitability, market share, and productivity. 

### Subheadings

Use three ### to include H3 headings.

Use **Bold** text for UI labels, use single back-tics for `parameters` and `filepaths`, and three back-tics for code blocks:

```
Write-Host "Hello, World!"
```

Use the following (minus the backtics) to include images:

```
![Alt text, a description of the image](/path/to/image.png "width=500")*Optional caption text*
```
If including images, please include alt text. Alt text is primarily used to describe images to people unable to see them, and can be 125 characters max including spaces. You can also include an image caption if the reader would benefit from additional information or context.

## Conclusion

Close off the post by restating the main points of the post, share any closing thoughts, and invite feedback.

## Learn more

- [link](https://www.example.com/resource)

## Register for the webinar: {webinar title here}

Short webinar description here, for example: A robust rollback strategy is key to any deployment strategy. In this webinar, we’ll cover best practices for IIS deployments, Tomcat, and full stack applications with a database. We’ll also discuss how to get the rollback strategy right for your situation. 

We're running 3 sessions of the webinar, from {webinar dates here, for example: 4 November to 5 November, 2021.}

<span><a class="btn btn-success" href="/events/rollback-strategies-with-octopus-deploy">Register now</a></span>

## Watch the webinar: {webinar title here}

<iframe width="560" height="315" src="https://www.youtube.com/embed/F_V7r80aDbo" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

We host webinars regularly. See the [webinars page](https://octopus.com/events) for details about upcoming events, and live stream recordings.

Happy deployments!
