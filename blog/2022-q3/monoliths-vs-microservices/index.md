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

A monolith is a singular executable software application. A monolith is one codebase that contains the user interface, application logic and database required to run an application. The application is built and maintained as a single, indivisible unit and served from a central location.

By contrast, a software application with a microservice architecture breaks down an application into smaller independent units. These units are decoupled and connected to make the whole application through Application Programming Interfaces (API). An API allows microservices to communicate to each other to execute a task. For example, a microservice sends data that is an input to another microservice. The second microservice sends input data to several other microservices, and so on. Microservices are modularized. The components that make up the application are built independently and only become associated with one another when required. A database component built as a microservice could be part of two different software applications through software reuse. Monoliths cannot be separated from the whole application without bringing application-specific code with it. This design decision is the key difference between monoliths and microservices.

## Diagram
<!-- Placeholder Image, get design to create a Octopus Image -->

<!--![Monolith vs Microservices](monolith-vs-microservices.jpg "width=500") -->

## Which one is better?

We have seen that a monolithic application is a single code base that describes everything an application needs to run. An application built from microservices has its codebase split across multiple components. These components are built independently and are connected on a case-by-case basis to suit application requirements.

Monolithic software is the traditional way of building software applications and has predated microservices for many years. The advancement of cloud technologies, containerization in particular, have allowed microservices to become very popular in delivering software. A Google Trends search shows that microservices started to become popular after 2013. This year is also when Docker (the main containerization technology) was released.

![Google Trends Microservices](google-trends-microservices.png "width=500")

Building, testing, and deploying a monolithic application involves the entire application. Troubleshooting a microservice would involve isolating, fixing, or swapping it out for another one. This makes microservices more agile than monolithic software. The agility of microservices allows them to be more reusable than monoliths. To reuse code from a monolith, developers need to extract the component and create new interfaces for the new application. Through APIs, microservices have immediate interfaces with other applications. This makes reconfiguring an application with microservices much easier than monoliths.

A container is a lightweight, portable computing environment with all the necessary binaries, configurations, and dependencies to run as a standalone process. Containerization is the process of making an application runnable as a container. Containerization has helped microservices gain popularity and widespread adoption through scaling. The application can leverage cloud orchestration tools like Kubernetes to schedule and scale workload at the component level. Suppose an application contains a database and a web front end, but the database is serving another application. In that case, a Kubernetes node can manage load at peak times for the web front end while increasing load to handle the database load for both applications. Compare this to scaling a monolith where increasing load for one component requires scaling all other components linearly.

## Challenges

Despite the many benefits, there are some challenges when using microservices over monoliths.

Microservices use API calls for communication. An application using microservices may have to make hundreds of API calls for one function. A monolith application would only need one call to do the same function. This leads to an increased latency to execute function calls. For applications that require several functions, monoliths can have higher performance at scale as there is no need for multiple API calls per function.

Both monoliths and the microservice architecture suffer from complexity issues. Developing a monolith means the code base of the application becomes very large. No single developer or team likely understands the application in its entirety. Microservices are independent, so each microservice is likely to be understood by its developers. Microservices also suffer from complexity issues because multiple services interact, and developers may not have applied a standard framework to each service. The complexity of microservices increases the demand for documentation as more services are added to the application. 

Microservice downtime can cause cascading effects in the application. Even though microservices are independent, they are connected. Often, several third party microservices are working together to achieve a task. If one microservice is sending incorrect outputs, this may cause other microservices to behave in an unpredictable way. Microservices are distributed, which introduces a networking problem. Firewalls, latency and security are larger problems than in microservices than monoliths where an application is self-contained, running in a central location.

## Conclusion

Monoliths and microservices are two different approaches to software development. Monoliths are a singular executable that provides high performance but lower agility. Microservices are interchangeable services that make up a larger product. Microservices provide high agility, flexibility and scale but increased networking and security considerations. The benefits of microservices make them better suited for enterprise software applications, they can bring software to market faster, and optimize infrastructure for cost.


!include <q2-2022-newsletter-cta>

Happy deployments!
