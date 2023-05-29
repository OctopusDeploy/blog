---
title: Building versus buying software
description: Probing the pros and cons of building software yourself compared to buying a product.
author: shawn.sesna@octopus.com
visibility: public
published: 2023-05-31-1400
metaImage: blogimage-build-vs-buy-2023x2.png
bannerImage: blogimage-build-vs-buy-2023x2.png
bannerImageAlt: Two people building with blocks vs someone at a storefront of a software store
isFeatured: false
tags: 
  - Engineering
---

Whether to build or buy software is a dilemma many organizations face. There's almost always a commercially-available tool or solution to address your needs. However, you must weigh up competing factors when deciding whether to buy or build to determine how much money *and* time it will cost.  

In this post, I explore what you should consider before deciding whether to build or buy a software solution.

## Considerations when deciding whether to build or buy

The considerations when deciding to build your own solution or purchase a product usually include: 

- Cost 
- Maintenance
- Support

### Cost

The price of a product can heavily influence your purchasing decision. Small organizations with a low budget often can't afford commercially-available solutions. This leaves them with no choice but to build something that suits their needs.  

Larger organizations need to weigh the cost of the solution at scale, as it needs to deliver value proportionally. If increased scale results in an exponential price increase, it may not be worth purchasing. 

Many paid solutions have moved from the traditional perpetual license to a subscription model. This means you need to include the cost of the product in your annual budget. 

Periodic price increases are common when the solution provider introduces more features. Sometimes you can choose to add improved features, or sometimes improvements are holistic, and you can't avoid a price increase. If you can't avoid the price rise, you need to evaluate whether the product gives you the same value it did when you chose it. 

Complicating this decision is the cost of switching to a built or competing solution. You need to consider the time and energy to onboard people to the new solution, training, and hardware purchases. Plus, additional cloud resources may be more expensive than the increased cost. Then again, the long-term savings may be worth it after you've made the switch.

#### Hidden costs

Cost doesn't just involve the price of a product. There are other costs associated with building your own solution. In-house solutions are sometimes considered "free" because people at your organization develop them, but these are usually false savings. While your team might have the technical ability, the cost to your organization can be more than the commercial product itself. You need to consider salary and benefits when determining the true cost of a custom solution. 

Time is another consideration. Building and maintaining a solution takes time from other projects that could benefit the company more than a "free" solution. 

However, one benefit of an in-house solution is no annual cost other than the time to properly maintain it. You can also customize your solution to your organization's needs and only include the features you require. A paid solution, however, might include features you never use.

### Maintenance

Maintenance of a solution is crucial to keeping it operating effectively. New versions of Operating Systems (OS) or development frameworks require you to test your solution to ensure it continues functioning. Providers of off-the-shelf solutions perform this activity to remain in business. If you build your own solution, you need to include time in your schedule to adapt your solution to changes.  

One potential pitfall of an in-house solution is that it doesn't get the attention it requires as it "just works" ... until it doesn't. Best case, you discover this early and allow time to resolve any issues. In the worst case, you find out at a crucial moment, leading to a mad scramble to fix it.

#### Security 

Security is another maintenance concern. Third-party libraries are often used in paid and in-house solutions to avoid reinventing the wheel. When someone finds vulnerabilities in third-party libraries, commercial companies react quickly to secure their software for their customers. 

You can include scanning software in your own solution, but it only runs when you build a new version. If you have a vulnerability, there's a window for exploitation until someone on your team discovers it. Source code scanning and runtime vulnerability monitoring tools can help mitigate this risk but require regular configuration and maintenance.

#### New features

As mentioned, companies that sell solutions regularly implement new features to keep up with evolving industry trends. Depending on your needs, this could be a benefit. When you decide to follow the trend, the feature is already there. If you have a purpose-built, home-grown solution, you may have to drastically modify your solution to keep up. For example, deploying an application to IIS differs vastly from deploying to Kubernetes.

### Support

Solutions that are critical to your organization need support. Often, if you have a successful, built solution, you employ people to support it or make it part of their jobs. In-house solutions sometimes become open-source projects that benefit from crowdsourcing to solve issues. The people supporting the solution can then help if the solution gets introduced at a different company. 

Open-source solutions also carry risks, though. Creators can abandon their solutions or move on, taking their expertise with them. To highlight this point, the popular CI server Jenkins has over [100 plugins](https://plugins.jenkins.io/ui/search/?labels=adopt-this-plugin) listed for adoption.

Conversely, paid products have support included in their pricing. The level of support varies, and you can sometimes buy higher levels of support. For example, a base product might offer support from 8am to 5pm, Monday to Friday. A premium support plan might offer 24/7 support at an extra cost. Your company policy might dictate which support plan you need. Some policies require all commercial products to come with 24/7 support. 

With a paid product, you always have someone to contact if something goes wrong.

## Conclusion

Early in my career, I wrote an automated deployment system that I implemented at 2 companies. One organization still uses the solution today, over a decade later. This shows that built solutions can be successful and offer a great return on investment. 

While the solution was successful at my second job, my role changed, and I had little time to develop new features or maintain it. A developer convinced me using a paid product was in the organization's best interest. After putting my pride aside, I implemented the paid solution... Octopus Deploy!

I hope this post helps you when deciding whether to build a solution or buy one. Both options have benefits and costs, but you must consider what's best for your organization.

Happy deployments!