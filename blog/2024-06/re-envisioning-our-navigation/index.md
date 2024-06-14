---
title: Re-envisioning our Navigation
description: A look into how we 
author: emily.pearce@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage: 
bannerImage: 
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags: 
  - tag
---

The find-ability of Octopus Deploy was our biggest UX pain point.  

Insights from our NPS surveys gave us hints about the navigation problem. But many comments were vague “i have trouble finding things” or “I feel lost”. This was a signal that it wasn’t a surface level problem or a quick fix.  

We came to learn many parts of the experience are connected to this problem. Information structure in the menus? Menu labels? Pages not needed? Page layouts? Is it where a user ends up once they finish a task? For us it was all the above.

So how do we tackle a broad and interconnected problem space? I decided to re-envision our navigation was the next best step. We organised a series of activities to help us understand our Navigation North Star.

## Gaining more understanding

_Card Sorting_

We gave existing customers a card sorting activity. This activity helps identify common information groups. Unfortunately, it didn't provide enough clarity. In particular, features for managing your deployments had produced conflicting results.  

_Mapping out mental models_

Next we wanted to see how we could better surface the mental model of Octopus in our navigation. We listed all the features of Octopus, then looked through different lens to group these features. This exercise brought better clarity. System scopes and personas bringing the most clarity.

-> insert image here
```
![Alt text, a description of the image](/path/to/image.png "width=500")*Optional caption text*
```


_Behavioural Analytics_

Looking at our analytics we obtained understanding of feature usage. Insights such as variable sets are heavily used. Yet, this feature was buried in our navigation.

_Ideation Workshops_

We brought the outcomes from mental model mapping, behavioural analytics and competitor research to help guide our first ideation workshop. It was clear that there could be a few ways to re-envision the navigation experience. 

## North Star
-> insert image here
```
![Alt text, a description of the image](/path/to/image.png "width=500")*Optional caption text*
```

Arriving at these rough concepts, we could now start validating our thinking and assumptions.  
Horizontal vs Vertical
Vertical tested slightly better than horizontal.

Menu Group Headings
Too many made it messy and over opinionated. Too little was overwhelming.  

Existing Customer vs New Customers
For DevOps professionals who had never used octopus before are able to find things with ease. Existing customers were very comfortable with receiving this type of UI change. 

### Noteworthy Learnings
+ No matter how great your navigation is, a poorly labeled feature will still confuse users. Yes Deployment Targets I am looking at you.
+ A navigation will only get you so far when you have 10,000s of things. Search and filters are the ideal tools. We gave global and in page search more importance in our designs.  
+ A point of comparison helped surface more understanding. Showing existing customers a new design enabled them to articulate what wasn’t working about the current design. 

## Delivering
Going with a North Star early on helped engineers plan activities to support the change. They identified foundational pieces of work that ensured making this change was relatively easily to implement.  We haven’t resolved all the find-ability problems and tradeoffs were certainly made. We will continue to refine our experience in a way that empowers customers to deploy on a Friday quickly and easily. 

```
![Alt text, a description of the image](/path/to/image.png "width=500")*Optional caption text*
```


## Learn more

- [link](https://www.example.com/resource)


## Watch the webinar: {webinar title here}

<iframe width="560" height="315" src="https://www.youtube.com/embed/F_V7r80aDbo" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

We host webinars regularly. See the [webinars page](https://octopus.com/events) for details about upcoming events, and live stream recordings.

Happy deployments!
