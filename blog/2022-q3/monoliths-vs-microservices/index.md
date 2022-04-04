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

Monoliths and microservices are two different approaches to building software. The choice between these two approaches can have significant impacts on development and performance. This blog discusses the difference between monoliths and microservices, identifies why microservices provide benefits over microservices for enterprise software applications, and lists some challenges to using microservices over monoliths.

## What are monoliths and microservices?

A monolith is a singular executable software application. A monolith is one codebase that contains the user interface, business logic, data access layer and database required to run an application. The application is built and maintained as a single, indivisible unit. In a production scenario, a monolithic application is served from a central location.

By contrast, a software application with a microservice architecture breaks down an application into smaller independent units. These units are decoupled from one another and are connected to make the larger application through several Application Programming Interfaces (API). An API allows two microservices to talk to each other. This could mean one microservice sending data that is an input to another microservice, which then outputs more data to other microservices. A software application that is built on microservices is modularized. This means that the components that make up the application are built independently and only become associated to another component when needed. A database component that is built as a microservice could be part of two different software applications depending on how those software applications are configured. The database component of a monolithic software application cannot be separated from the overall larger application without bringing application specific code with it. This design decision is the key difference between monoliths and microservices.

## Diagram
<!-- Placeholder Image, get design to create a Octopus Image -->

<!--![Monolith vs Microservices](monolith-vs-microservices.jpg "width=500") -->

## Which one is better?

We have seen that a monolithic application is a single code base that describes everything an application needs to run. A application built from microservices has its code base split across multiple components. These components are built independently and are connected together on a case by case basis to suit application requirements.

A monolithic software architecture is seen as the traditional way of building software applications. In practice, microservices were not feasible until after monolithic applications had been built and delivered for many years. In recent years, microservices have become very popular in delivering software. The main reasons for this have been containerization and cloud as-a-service technologies enabling microservice concepts. A quick Google Trends search show that microservices as a term only started to become popular after 2013 (coincidentally, when Docker got released).

![Google Trends Microservices](google-trends-microservices.png "width=500")

Monolithic software is not as agile as microservices. Building, testing, and deploying a monolithic application involves the entire application. Troubleshooting a deployment in a microservice would involve isolating the microservice in question and either fixing it or swapping it out for one that has already been proven to work by itself.

Increased agility leads to the advantage of reusability that microservices have over monoliths. The database that is built for a monolithic application is not designed for reuse. The code and functionality in the monolithic codebase could be reused for a later project by extracting and creating new interfaces for the new application. But compared to microservices, this is not true reusability. Microservices are designed to be used as a component in several different software configurations rather than being designed as part of a larger singular whole like monoliths. This means that a  microservice component can interface with any other microservice through APIs and it is the APIs that allow developers to swap an old component with a new one with minimal fuss.

We discussed how containerization enabled microservices to gain popularity and widespread adoption. A key benefit that containerization allows is scaling. If every microservice that comprises an application can be containerized, the application can leverage cloud orchestration tools like Kubernetes to schedule and scale workload at the component level. If an application contains a database and a web front end, but the database is serving another application, a Kubernetes node can manage load at peak times for the web front end while at the same time increase load to manage the data base load for both applications. Compare this to scaling a monolith where increasing load for one component requires scaling all other components linearly.

## Challenges

Despite the many benefits, there are some challenges to using microservices over monoliths.

Microservices are built on API calls. A application using microservices may have to make hundreds of API calls for one function. A monolith application would only need one call to do the same function. At scale, monoliths can have higher performance as there is no need for multiple API calls per function.

Complexity is often cited as advantage of microservices over monoliths. Developing a monolith means the code base of the application becomes very large. It is likely that no single developer or team understands the application in its entirety. Microservices are built to be independent, and so each microservice is likely to be understood by its developers. Applications using microservices also suffer from complexity issues because there are multiple services interacting and there may not have been a standard framework applied to each service.

The interrelated nature of microservices mean that downtime can cause cascading effects in the application. If one microservice is depending on an output of another microservice, and so on, troubleshooting becomes a cross-service problem rather than a contained problem with a monolith.

The distributed nature of microservices introduces maintenance problems that are not present in monoliths. Due to the multiple API calls of microservices, this introduces a networking and security problem. Latency would be an important metric to minimize as well as securing the communication between microservices via API calls.  

## Conclusion

Monoliths and microservices are two different approaches to software development. Monoliths are a singular executable that provide high performance but higher complexity and lower agility. Microservices are a collection of services that make up a complete product. They provide better flexibility and scalability compared to monoliths. The flexibility and scalability of microservices make them more suitable then monoliths for enterprise software applications.


!include <q2-2022-newsletter-cta>

Happy deployments!
