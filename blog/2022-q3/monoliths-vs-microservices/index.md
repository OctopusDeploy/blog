---
title: Monoliths vs Microservices
description: A brief summary of the post, 170 characters max including spaces.
author: terence.wong@octopus.com
visibility: private
published: 3020-01-01-1400
metaImage:
bannerImage:
bannerImageAlt: 125 characters max, describes image to people unable to see it.
isFeatured: false
tags:
  - DevOps
  - Runbooks Series
  - Runbooks
---

<!-- see https://github.com/OctopusDeploy/blog/blob/master/tags.txt for a comprehensive list of tags -->

Monoliths and microservices are two different approaches to building software. The choice between these two approaches can significantly impact development and performance. This blog discusses the difference between monoliths and microservices, identifies why microservices provide benefits over monoliths for enterprise software applications and lists some challenges to using microservices over monoliths.

## What are monoliths and microservices?

A monolith is a singular executable software application. A monolith is one codebase that contains the user interface, business logic, data access layer, and database required to run an application. The application is built and maintained as a single, indivisible unit and served from a central location.

By contrast, a software application with a microservice architecture breaks down an application into smaller independent units. These units are decoupled and connected to make the whole application through Application Programming Interfaces (API). An API allows microservices to communicate to each other to execute a task. For example, a microservice sends data that is an input to another microservice. The second microservice sends input data to several other microservices, and so on. Microservices are modularized. The components that make up the application are built independently and only become associated with another component when needed. A database component built as a microservice could be part of two different software applications by reusing the database component twice. Monoliths cannot separate the database component of a software application from the whole application without bringing application-specific code with it. This design decision is the key difference between monoliths and microservices.

## Diagram
<!-- Placeholder Image, get design to create a Octopus Image -->

<!--![Monolith vs Microservices](monolith-vs-microservices.jpg "width=500") -->

## Which one is better?

We have seen that a monolithic application is a single code base that describes everything an application needs to run. An application built from microservices has its codebase split across multiple components. These components are built independently and are connected on a case-by-case basis to suit application requirements.

Monolithic software architecture is the traditional way of building software applications. Microservices were not feasible until monolithic applications existed for many years. In recent years, microservices have become very popular in delivering software. The main reasons for this have been containerization and cloud as-a-service technologies. A quick Google Trends search shows that microservices only started to become popular after 2013 (coincidentally, when Docker got released).

![Google Trends Microservices](google-trends-microservices.png "width=500")

Monolithic software is not as agile as microservices. Building, testing, and deploying a monolithic application involves the entire application. Troubleshooting a microservice would involve isolating, fixing, or swapping it out for another one.

Increased agility leads to the advantage of reusability that microservices have over monoliths. Monolithic applications are not reusable. Developers could use the code and functionality in the monolithic codebase for a later project by extracting and creating new interfaces for the new application. But compared to microservices, this is not true reusability. You can find microservices in several different software configurations rather than being designed as part of a larger singular whole like monoliths. A  microservice component can interface with any other microservice through APIs. The APIs allow developers to swap an old component with a new one with minimal fuss.

We discussed how containerization enabled microservices to gain popularity and widespread adoption. A key benefit that containerization allows is scaling. The application can leverage cloud orchestration tools like Kubernetes to schedule and scale workload at the component level. Suppose an application contains a database and a web front end, but the database is serving another application. In that case, a Kubernetes node can manage load at peak times for the web front end while increasing load to handle the database load for both applications. Compare this to scaling a monolith where increasing load for one component requires scaling all other components linearly.

## Challenges

Despite the many benefits, there are some challenges to using microservices over monoliths.

Microservices use API calls. An application using microservices may have to make hundreds of API calls for one function. A monolith application would only need one call to do the same function. Monoliths can have higher performance at scale as there is no need for multiple API calls per function.

Complexity can be an advantage of microservices over monoliths. Developing a monolith means the code base of the application becomes very large. No single developer or team likely understands the application in its entirety. Microservices are independent, so each microservice is likely to be understood by its developers. Applications using microservices also suffer from complexity issues because multiple services interact, and developers may not have applied a standard framework to each service.

The interrelated nature of microservices means that downtime can cause cascading effects in the application. Because of the communication between microservices, troubleshooting becomes a cross-service problem rather than a contained problem with a monolith.

The distributed nature of microservices introduces maintenance problems not present in monoliths. Due to the multiple API calls of microservices, this introduces a networking and security problem. Latency would be an essential metric to minimize to improve performance. The high volume of API calls makes it necessary to secure communications between microservices.

## Conclusion

Monoliths and microservices are two different approaches to software development. Monoliths are a singular executable that provides high performance but higher complexity and lower agility. Microservices are a collection of services that make up a complete product. They offer better flexibility and scalability compared to monoliths. Microservices are a better choice than monoliths for enterprise software applications.


!include <q2-2022-newsletter-cta>

Happy deployments!
